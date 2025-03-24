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


|Index|Output|
|---|---|
|  plt.imshow(np.flip(array[0, :, :])) | ![image](https://github.com/user-attachments/assets/2a8e0e20-60a4-411d-ae7d-4531fd7f1b76) |
|  plt.imshow(np.flip(array[1, :, :])) | ![image](https://github.com/user-attachments/assets/cec5818c-1c58-4e53-991f-7ae678ffb560) |
|  plt.imshow(np.flip(array[2, :, :])) | ![image](https://github.com/user-attachments/assets/0121a06f-c5e8-499d-b08b-870181a90b01) |
|  plt.imshow(np.flip(array[3, :, :])) | ![image](https://github.com/user-attachments/assets/fe55fd4b-6144-4525-84b8-de802d47688f) |


