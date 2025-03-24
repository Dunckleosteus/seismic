# Intro
The goal of this project is to provide a 2D synthetic seismic panel from velocity and density data on a carbonate outcrop. 
# Data 
An excel file split into 4 sheets: 
- X
- Y $\rightarrow$ z axis
- VP
- RHOB
# Code 
## Python import
The choice was made to run the code on google collab and store version on github to allow for collaboration and versioning of code. Keeping that in mind, the excel sheel was saved to 4 different csv files stored within the seismic zip file. 
To install the files directly in the current notebook from collab, we use the following bash commands : 
```shell
!rm data.zip
!rm -r data
!wget https://github.com/Dunckleosteus/seismic/raw/18908e9e23b56529c5744c96fac2a5216cd298cd/data.zip
!unzip data.zip
```
The unzipped file will then be available @ path: `data/*.csv`. The paths are added to a list in python, allowing them to be opened iteratively. 
> The list could have been replace by a glob command as well

```python
df_path_list = [
    "data/rhob.csv",
    "data/vph.csv",
    "data/x.csv",
    "data/y.csv",
]
```
A list of dataframes can then be created via list comprension. 
```python
df_list = [pd.read_csv(path, index_col=None, header=None, sep=";") for path in df_path_list]
```
A replace function within a try catch statement is then used to replace commas with point to allow for conversion into numeric format. 
```python
[df.dropna(inplace=True) for df in df_list]
for df in df_list:
  for col in df.columns:
    try:
      df[col] = df[col].str.replace(",", ".").astype(float).values
    except:
      df.values
```
The values of the data frame are then stacked, similarly to a multiband image. This facilitates computation by leveraging the numpy vectorization capabilities. 
```python
array = np.stack(df_list)
array.shape
```
The various "bands" can then be visualized with numpy indexing as illustrated in the table below : 


|Name|Index|Output|
|---|---|---|
| RHOB | plt.imshow(np.flip(array[0, :, :])) | ![image](https://github.com/user-attachments/assets/2a8e0e20-60a4-411d-ae7d-4531fd7f1b76) |
| VPH | plt.imshow(np.flip(array[1, :, :])) | ![image](https://github.com/user-attachments/assets/cec5818c-1c58-4e53-991f-7ae678ffb560) |
| X | plt.imshow(np.flip(array[2, :, :])) | ![image](https://github.com/user-attachments/assets/0121a06f-c5e8-499d-b08b-870181a90b01) |
| Y | plt.imshow(np.flip(array[3, :, :])) | ![image](https://github.com/user-attachments/assets/fe55fd4b-6144-4525-84b8-de802d47688f) |

## Accoustic impedance
The formula for accoustic impedance is :
$$I_i=\rho_i \times v_i$$

So to calculate it, we simply need to multiply the first layer (np.flip(array[0, :, :])) in array to the second one (np.flip(array[0, :, :])).
```python
impedance_matrix = array[0, :, :] * array[1, :, :]
plt.imshow(impedance_matrix)
plt.title("Accoustic impedance")
```
The result is as follows: 

![image](https://github.com/user-attachments/assets/23d26aad-7f7e-4e5e-a19a-6082bba6e3b2)
> Note that it's basically a copy of VPH because RHOB is constant.

## Conversion to time domain 
Conversion from distance to time was acheived with the following code : 
```python
time_matrix = (0.2 / array[1, :, :])
time_matrix = np.cumsum(time_matrix, axis=0) * 2
```

## Reflection coefficient
$$R_o=\frac{I_{i+1}-I_i}{I_{i+1}+I_i}$$
Where I is the impedance calculated above.

To acheive this in python, we use the `np.roll` tool to create a second matrix offset vertically : 
```python
I     = np.roll(impedance_matrix, shift=1, axis=0)[:-1, :] # n+1
In1  = impedance_matrix[1:, :] # n
reflection_coeff = (In1 - I) / (In1 + I)
plt.imshow(reflection_coeff)
```
Given two matrices, one row is removed from both the top and bottom of each matrix. After this trimming, the remaining pixels are compared pairwise across corresponding rows and columns to analyze the differences or patterns between them. The result is presented in picture below: 

![image](https://github.com/user-attachments/assets/dcb81bd3-e252-4f7e-87df-5b756caf2495)

Compared to previous plots, the reflection coefficient highlights differences in accoustic impedance between cells. The vertical differences in each column vertically can be represented vertically with a fishbone plot : 

![image](https://github.com/user-attachments/assets/d2ff703a-2986-4f64-9853-df4845bf1358)


The plot was obtained with the following code : 

```python
plt.stem(reflection_coeff[:, 1], orientation='horizontal')
plt.gca().invert_yaxis()
```
Were we can see, from the numpy index, that it represents the second row column from the left.

# Wavelet
The goal here is to create a wavelet to convolve with the reflectivity data. To this end the bruges package is used. To install it in collab : 
```
!pip install bruges
```
Then : 
```python
import bruges.filters as bf
```
## Creating wavelet
Following this, a function was created to facilitate wavelet creation : 
```python
def create_wavelet(offset, amplitude, length=0.256, frequency=40, sample_intervall=0.01):
  w, t = bf.ricker(length, sample_intervall, frequency)
  t += offset
  w *= amplitude
  return w,t
```
... 
whith which a wavelet was defined and plotted :

![image](https://github.com/user-attachments/assets/36927dc1-d3d8-4383-b16c-7e84995c26fd)

With the following code: 
```python
w, t = create_wavelet(1.5, 0.2, sample_intervall=0.001)
plt.plot(w, t)
plt.scatter(w,t )
```
## Wavelet convolution
To convolove the wavelet with the fishbones, 2 functions were defined : 
```python
def expand_with_zeros(arr, target_length): #fill des valeurs manquantes par des 0 pour ne pas modifier la données
    if len(arr) == target_length:
        return arr
    new_arr = np.zeros(target_length)
    indices = np.linspace(0, target_length - 1, len(arr), dtype=int)            # create correct size matrix of 0
    for i, index in enumerate(indices):                                         # fill the matrix with values of initial matrix @ correct index
        new_arr[index] = arr[i]
    return list(new_arr)

```
And 
```python
def equalize_arrays(arr1, arr2): #faire en sorte que les deux arrays aient le meme nombre de points
    max_length = max(len(arr1), len(arr2))

    arr1_expanded = expand_with_zeros(arr1, max_length)
    arr2_expanded = expand_with_zeros(arr2, max_length)

    return arr1_expanded, arr2_expanded
```
They are used to ensure that wavelet array is of compatible dimension with fishbone so as to use `np.convolve` function. Here is application : 
```python
rc2, w2 = equalize_arrays(rc, w)
syn = np.convolve(rc2, w2, mode='same')
```

These steps are then implemented into a for loop to apply it to each column of reflectivity matrix. Here is an output example for a single column : 

![image](https://github.com/user-attachments/assets/664416a4-4762-44bb-8df2-1f096f1885b5)

Applied to all outputs, the results are as follows : 

![image](https://github.com/user-attachments/assets/f3d5d126-0eed-4dfd-9c6f-bb95886de219)


