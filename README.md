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

## Reflection coefficient
$$R_o=\frac{I_{i+1}-I_i}{I_{i+1}+I_i}$$
Where I is the impedance calculated above.

This operation is 1D and is deployed on vertical axis. Here are vertical axis is represented by y.

