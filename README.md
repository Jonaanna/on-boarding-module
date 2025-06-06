# onbording-base-module - F1 Praktikum
 *Jona Hofmann 5.05.2025*

## Short Introduction 
This project focuses on learning the basics of quantitative image analysis techniques using biomedical data. Key bioinformatics skills are covered, including setting up a development environment with **Visual Studio Code**, creating a project structure, and using **Git** for version control.

*In this file I will document the steps taken to complete the tasks, along with background information for further understanding.*

## Biological Background / Content
- Determination of spatial localization of transcripts from 50 mouse genes  
- Use of a published [in situ sequencing dataset](https://www.ebi.ac.uk/biostudies/bioimages/studies/S-BSST700) including barcodes from four rounds of sequencing  
- Counting cell numbers and gene expression per field 

## Biological Data Description

- *What is the data?*
    Selected tiles of the In Situ Sequencing Mouse Brain Dataset NT_ISS_KR0018
- *What files are used?*
    ZIP file format is used including `.csv`, `.png`and `.hs` files
- *How to Access the Data?*
     Download the dataset from  [BioImage Archive](https://www.ebi.ac.uk/biostudies/bioimages/studies/S-BSST700)
- *What is the data about?*
    **Channel info** Puts the channel together with its code 
    **taglist**  barcodes used in this experiment
    **Name of tiles** The dataset consists the names of  tiles.
    **map of selected tiles:** Contains a map of the selected tiles on the mousebrain
    **selected tiles** The dataset includes the images of the selected tiles from the mousebrain
    **Decoding** Decoding of ISS tiles 



## Work Plan – Tasks - Installing/PreWork
### 1. Read the entire script to gain a clear understanding of the topic  
### 2. Install [Visual Studio Code](https://code.visualstudio.com/) as an integrated development environment  
   - Useful for programming due to features like terminal access, Git integration, debugger, etc.
### 3. Create a standardized project folder structure to maintain organization  
### 4. Install [Git](https://git-scm.com/) for version control  
### 5. Create this `README.md` file  
### 6. Download dataset from [BioImage Archive](https://www.ebi.ac.uk/bioimage-archive/)  
   - ZIP download causes issues due to large size → alternative: download via FTP  
### 7. Create `.gitignore` file for ignoring specific files that should not be tracked -> include `data/`folder
### 8. Update this `README.md` file 
### 9. For a first understanding look at the data visually
### 10. Install a package manager to work with the data
-> Miniforge3 = installer for Mamba  
### 11. Create an environment `myenvironment.yml`file. In this file put all the packages you want to install like:  
  - numpy
  - pandas=2.0.1
  - python=3.11
  - matplotlib
  - scikit-image
  - ipykernel
- commit `myenvironment.yml`to git with : `git add environment.yml` and update this file
### 12. Install Jupyter in the extensions 
*Why Jupyter when we already have an environment?* 

    Jupyter is a powerful tool when paired with VSC, particularly for visualizations and analysis. It is ideal for interactive data analysis

## Work Plan - Tasks - Coding
### 13. Figure with all subplots (c1-c4)
#### 13.1. Plot the image `out_opt_flow_registered_X10_Y10_c01_DAPI.tif`:
 -> import libraries with `import ... as ...`

 -> read the image with the right path! 

 -> save the image in a variable with `image = io.imread(image_path)`

 -> show image

*UPDATE: git + README.md* : ✅

#### 13.2. Create a list of files with all X_10_Y10 images
 -> Show all files of 'selected-tiles folder' with `glob.glob(file paths)` as a list : to select the ones with X_10_Y10 type * after the path

 -> save the selected files with `io.imread()` as a list of numpy-arrays and show them with `print()`

 *UPDATE: git + README.md* : ✅

#### 13.3. Plot the X_10_Y10 images shown as a grid
-> Sort all images by pattern: name of dye (=column) and number of tile  c01-c04 (=row)
1. for example "Alexa_488" for dye and "c01" for one tile
2. Load the list
3. Search for the selected image files with the two pattern by creating a loop:
 Searching for maching file - loading and storing the image - saving the image file name for labeling

-> Create a grid of 4 x 6 of the images with a loop

```python
# Show the pictures in a grid with 4 rows and 6 colomns
fig, axs = plt.subplots(nrows=4,ncols=6, figsize=(25,15))

for i, ax in enumerate(axs.flatten()): # loop for each subplot (ax) in the grid #flats list for loop
    ax.imshow(image_list[i]) # i = index for all images in image_list ax = plot object for every image
    filename = os.path.basename(file_list[i])
    ax.set_title(image_names[i], fontdict={'fontsize': 9, 'fontweight': 'bold', 'fontname': 'Arial'}) # adds title
plt.show() # shows image
```
#### Create a function in `Functions.py`
-> Create the function plotImage(): 

-> Try the function in `Analysis.ipynb`

*UPDATE: git + README.md* : ✅

### 14. Nuclei count in each field of view (fov)

**INFOS form data** 
-> Open channel_info.csv to decide the annotation for all channels

### Channel Information

| Channel       | Label     | Purpose             | Suitable for Nucleus Segmentation 
|---------------|-----------|---------------------|---------------------------------- 
| DAPI          | nuclei    | Stains DNA/nuclei   | ✅ Yes                            
| Atto_490LS    | anchor    | Image aligment      | ❌ No                             
| Atto_425      | A         | ISS base (Adenine)  | ❌ No                             
| Alexa_488     | C         | ISS base (Cytosine) | ❌ No                             
| Alexa_568     | G         | ISS base (Guanine)  | ❌ No                             
| Alexa_647     | T         | ISS base (Thymine)  | ❌ No                             
nCycles = 4
nChannel = 6

**Which channel(s) is/are most promising for nucleus segmentation?**

-> DAPI is labeled for `nuclei` and therefore the most promising for nuclei segmentation. It binds the DNA, especially in the cell nucleus.

**Which methods can you use, to distinguish between nucleus and the rest?**

*preparation*
-> Import selected tile (channel DAPI)
-> Convert the image to grey
-> show the pixels in a histogram to see the range of the pixels with `.flatten().shape` and `plt.hist()`
-> Blurring with `ski.filters.gaussian()` to reduce noises, improve thresholding

*Threshold set manual*
-> set the threshold by yourself: minimal and maximal pixel value helps for choosing the trheshold

*Automatised threshold with Otsu*
-> import `trheshold_otus from `skimage.filters

*Binary mask*
-> Instead of thresholds you can create a binary mask 

*count nucleus*

**How can you devide the nuclei into eperate instances?**

*Watershed*
-> to use watershed for seperating nuclei it is important to create a mask of the local maxima and minima. For the mask you need to create a map where you have the nuclei as highest value and background as lowest

    - `ndi.distance_transform_edt()` for map
    - `feature.peak_local_max()` for the coordinates
    -  `np.zero_like` for the empty_mask
    -  local max = True to get max in mask 
    - `ndi.label()[0]`to label the max

-> `segmentation.watershed()`

**Problems that accure while counting nuclei in fov**

-> The images could have small artifacts which could be seen as nuclei 
    (removing small objects)

-> Cell nuclei can be close to one another which could lead to overlappig nuclei, that are detected as only one
    (seperation with wathershed)

-> wrong threshold leads to background noice 
    (try different typtes of threshold like Otsu threshold or manual, decide which seems more accurate)

-> every image is individual, not every problem can be addressed the same way
    (try your method on different image)

**Try the method on other fov**

-> create a list with the file paths of your images*
-> use your method to count the nuclei and show the images with detected nuclei

## 14.2

**Script for your method on all fovs**

-> use `glob.glob()` to search for all files with `*DAPI*` in it 

-> Write the count per fov to a csv file
```python
with open("nuclei_count.csv", "w", newline="") as csvfile:
# open the file, (name of file, w for write, newline to begin a new line after something ws added), save as csvfile

    writer = csv.writer(csvfile) # to write in this script
    writer.writerow(["image name", "nuclei counted"]) #header for the "table" that follows
```
-> Create a loop and put the method in the loop of the CSV-file

-> save the plots as png files in a folder
```python 
output_folder = "diagnostic_plots"
os.makedirs(output_folder, exist_ok=True)
```

-> count all nuclei 

#### extra for 14.2

-> save the fov, with mean and standard devision into one table
-> show a plot with the mean counted nuclei for all fov and add mean and sd 

#### problems accured 

->  Counted nuclei on tile X23_Y4 showed unexpected results (arount 8000 counted nuclei on c1); Tile should not be in selected tiles (deleted)


### 15. Find and identify the transcripts

#### 15.1 Spot detection 

-> Detect the spots of the channels with bases (A,C,G,T) in all four rounds 
 
 1. Create a loop that goes through your selected tiles folder and skips all unwanted channels (DAPI and Atto_490LS)

 2. You need do organize the files with FOV detection for X and Y, Cycle detection and base netection

3. Structure for the dictonary with Field of view (X_Y), the cycle and the base 
  -> You will get a dictonary with the right structure: 
  ```python 
  'X10_Y10': {0: {'A': '/Users/jonahofmann/onboarding-base-module/data/selected-tiles/out_opt_flow_registered_X10_Y10_c01_Atto_425.tif',
                 'C': '/Users/jonahofmann/onboarding-base-module/data/selected-tiles/out_opt_flow_registered_X10_Y10_c01_Alexa_488.tif',
                 'G': '/Users/jonahofmann/onboarding-base-module/data/selected-tiles/out_opt_flow_registered_X10_Y10_c01_Alexa_568.tif',
                 'T': '/Users/jonahofmann/onboarding-base-module/data/selected-tiles/out_opt_flow_registered_X10_Y10_c01_Alexa_647.tif'}
```

4. Spot detection with a loop that iterates over each FOV 

    -> Use  `filters.difference_of_gaussians()` to highlight get a better detection of  the spots

    -> Use `peak_local_max` as threshold and adjust the distance to get no background noises

    -> show example for spot detection

#### 15.2  Barcode decoding
**What happens at in-situ sequencing:** 

-> Cycle of fov has 4 possible channels A;C;G;T and every detected spot means one base at one position inside the mousebrain 
-> the following cycle will have a spot at the same position, with another base 

---> If you follow one spot over all cycles you can get the complete sequence 

1. Create a loop and iterate through every fov, then every base, to connect the spot in a cycle

2. Follow the detected spot of the cycle trhough all cycles, if you can't find the base say 'N', if you find the base:
    -> You need to check if the spot is within max_distance of the previous spot with `cKDTree()`
    -> save the position of the base for the next cycle 

3. Compare your results of the the barcode for a spot with `taglist.csv` and assign unknown barcodes to "invalid" so you can then count the number of all detected sequences 

4. Save the file  with the barcode decoding results in a csv file 


**Problems that can accure**

*Possible complications are cases, where no channel or multiple channels have high intensity within the same round of sequencing.*

    -> Filter the spots as 'invalid' and ignore them (only for now)
    -> instead of deleting these channels it is possible to detect them with a flag 
    -> Improve the quality of the images by reducing noises and optimizing the threshold
    -> machine learning and statistics 

### 15.3 Visualize the results in napari 

1. Create a script called `Visualized.py` where you put your code 
    -> you can then run the code through the terminal with `python "path"`

*The code should contain:* 

- Opens  all images of one FOV in napari with all four cycles 
    -> don't forget to install the packages, write napari into your `myenvironment.yml` file 
- seperate naapari window for each FOV
- create layers for all four images of sequencing channel 
- load the base specific channels A;C;G;T and add colors 
- add the decoded spots to the napari viewer as a points layer with `barcode_decoding_results.csv`
- Add DAPI images if availabe

### 15.4 All fovs
**Run spot detection and barcode decoding for all fovs, creating a single csv file with predictions for all fovs**

- load the barecode_decoding_results.csv: create with a loop through all decoded sequences

```python 
barcode_decoding_results = []
for fov, x, y, barcode in all_sequences:
    if barcode.count("N") > 1:
        gene = "invalid" # assign unknown barcodes to invalid
    else:
        gene = barcode_to_gene.get(barcode, "invalid") # get the gene from the barcode or assign invalid if not found
    barcode_decoding_results.append({
        "fov": fov,
        "x": x,
        "y": y,
        "barcode": barcode,
        "gene": gene
    })
```
 ### 15.4 All fovs
**Create another csv file with aggregated counts for each gene and fov: fov, gene, count**
- group the data by FOV and gene 
- count with `.size()`how many spots exist for each fov and gene 
- .size().reset_index(name="count") gives new dataframe with colomns fov, gene and count 
- save as `gene_counts_by_fov.csv` file 

### 15.5  Compare your results

 The downloaded data provides a folder called `decoding.zip`. There you can find different methods that created spot location and decodings.

 - How many **spots in total** have you counted compared to the other methods?
    - First its important to look at the provided files and check the Colomn names. If they differ you have to keep that in mind --> (gene - Name; fov - Tile)
    - Also filter invalid entries like "invalid" and "infeasible" ,...
    - count the spots and combine the counts
    - plot the results
    -> compare argmax, kmeans, poSTcode and myMethod

 - Compare the relative mRNA percentage per gene: posSTcode vs. MyMethod 
    - Plot all genes of both methods cand compare the outcome
    - What is the most/least frquent gene in myMethod and PoSTcode
    - Largest percentage difference with PoSTcode  
