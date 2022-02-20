# Match Roads Between Databases
I independantly wrote a program to match roads sections from different traffic databases using geographic information.
## Background
My supervisor at the University of Toronto Transportation Research Institute was involved in the calibration of a traffic simulation model covering the GTHA to real world conditions. This means modifying simulation parameters until conditions and traffic flows match those given by third party sources. Given observed data from HERE techologies, my supervisor wanted to know which sections in the simulation model were being described.
## Problem
- over 100,000 road sections per database (strong time complexity requirements)
- naming is inconsistent
- location of road section splits are inconsistent between databases
- close proximity does not gaurantee correct match
- geographic coordinates carry up to 10 metres of uncertainty

## Data
Data is given as 2 GeoPandas dataframes, with each row describing a section. Information includes road name, connected sections, and geometric characteristics given as shapely objects. In total there are 160,000 and 300,000 sections per database.

![Visualization of road sections in Database 1](images/ex1_HERE.png) ![Visualization of road sections in Database 2](images/ex1_aimsun.png) ![Visualization of query road and returned matches](images/ex1_match_background.png) 
## Architecture
My program consists of the following stages. 
1. Data is reorganized into a new data structure that allows fast queries based on geographic location.
2. Algorithm matches roads between databases.
3. Visualizations and statistics regarding matching process integrity are generated.
### Data Reorganization
In the original Pandas dataframe, searching for a road section near coordinates is O(N) time. With N queries this gives us O(N^2) time, or roughly 10^10 steps. I restructured key data into a stack of ordered lists, which allows for binary search (figure below). Each query takes approximately O(log(N/nl)) time, where nl = number of lists = 8000. N queries run in roughly 5*10^6 steps.

![Visualization of road sections in Database 1](images/data_reorganization.png)

### Matching Algorithm
The matching algorithm works primarily with geo-spatial information. Because separate roads commonly overlap or are close, we need a better algorithm than a simple distance heuristic. My solution combines a distance heuristic, with connected section information to filter out most incorrect matches.

Step 1: Find sections near each endpoint of the query road, and store them in 2 sets.

![Visualization of road algorithm matching process Step 1](images/ex2_endpoints_blue.png)

Step 2: We run a search algorithm to find a path connecting the 2 sets. If a path is found, and all sections along the path also lie close to the query road, then this path should be the query road itself.

![Visualization of road algorithm matching process Step 2](images/ex2_matches.png)
### Statistical Visualization
Finally, the overlap distance and direction of each matched section to the query road is calculated via geometric manipulations of shapely objects. Optional visualizations are produced on every process to test program integrity.

![Visualization of road sections in Database 1](images/overlap_calcs.png)
## Results and Discussion
### Accuracy
94% per query when excluding freeways and sections shorter than 10 metres. 85% otherwise. Accuracy is measured as having all matches of the query road being correct. Note that accuracy is highly dynamic as overlap information and other parameters can be used as an adjustable threshold to improve results.

### Improvements
#### Data Organization
Instead of creating my own datastructure to allow for faster coordinate based search queries, I would explore libraries that organize data according to 2 dimensions for me. This will make my code simpler and more reproducible.
#### Matching Algorithm
I would explore utilising secondary information (ex. overlap distance, road names, road direction) as a second threshold to improve accuracy. Since the program would now have 2 distinct filters, accuracy is expected to be very high.
\
\
\
Skills Learned: GeoPandas, GIS, shapefiles, shapely visualizations, matrix/array operation time dependancies
