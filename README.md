# Understanding the Influence of Parameter Value Uncertainty on Climate Model Output: Developing an Interactive Dashboard
This README.txt file was generated on 2024-05-22 by SOFIA INGERSOLL 
GENERAL INFORMATION
1. Title of the Project: Understanding the Influence of Parameter Value Uncertainty on Climate Model Output: Developing an Interactive Dashboard
2. Author Information: Sofia Ingersoll
A. Principal Investigator Contact Information Name: Sofia Ingersoll Institution: Bren School of Environmental Science & Management Address: Email: singersoll@ucsb.edu
B. Associate or Co-investigator Contact Information Name: Heather Childers Institution: Bren School of Environmental Science & Management Address: Email: hmchilders@ucsb.edu
C. Alternate Contact Information Name: Sofia Ingersoll Institution: Bren School of Environmental Science & Management Address: Email: Sofia.Ingersoll@Outlook.com
3. Date of data collection or obtaining (2024-05-23) 
4. Geographic location of data collection: University of California, Santa Barbara – Bren School of Environmental Science & Management.  
5. Information about funding sources that supported the collection of the data: N/A
SHARING/ACCESS INFORMATION
1. Licenses/restrictions placed on the data:
This data is open source and available for climate scientists to leverage for climate model calibrations and identification of key climate influencers. 
2. Links to publications that cite or use the data: N/A
3. Links to other publicly accessible locations of the data:
Visualize the data in this repository using an interactive dashboard. The dashboard enables public access to the National Center for Atmospheric Research – Climate and Global Dynamics Lab: Parameter Perturbation Experiment (NCAR – CGD: PPE) Data. The PPE was generated by NCAR – CGD at the University of California, Santa Barbara. [Link to public facing dashboard will be added here once client integrates into server]
4. Links/relationships to ancillary data sets:
Visual example of the Community Land Model v5 (CLM5) climate model components and their respective parameter interactions. https://www.cesm.ucar.edu/models/clm
Table of output variables for the Community Earth Systems Model v2 (CESM2) Large Ensemble Community Project. CESM2 is the climate model that supersedes the CLM5. The predictions provided for the CESM2 output variables were leveraged by the CLM5 to produce its predictions. The variables available in CESM2 are accessible using our dashboard linked above.
https://www.cesm.ucar.edu/community-projects/lens2/output-variables
Table of specifications for the Community Land Model v5 (CLM5) https://www.cesm.ucar.edu/models/clm/data
5. Was data derived from another source? If yes, list source(s): 
University of California, Santa Barbara – Climate and Global Dynamics Lab, National Center for Atmospheric Research: Parameter Perturbation Experiment (CGD NCAR PPE-5). https://webext.cgd.ucar.edu/I2000/PPEn11_OAAT/
The Parameter Perturbation Experiment data leveraged by our project was generated utilizing the Community Land Model v5 (CLM5) predictions. https://www.earthsystemgrid.org/dataset/ucar.cgd.ccsm4.CLM_LAND_ONLY.html
6. Recommended citation for the project:
Ingersoll Sofia, Childers Heather, Bhattarai Sujan. “Understanding the Influence of Parameter Value Uncertainty on Climate Model Output: Developing an Interactive Dashboard”. (2024). University of California, Santa Barbara – Bren School of Environmental Science & Management. https://github.com/GaiaFuture/CLM5_PPE_Emulator
DATA & FILE OVERVIEW
1. File List: <list all files (or folders, as appropriate for dataset organization) contained in the dataset, with a brief description of their content>
For multiple datasets sharing the same characteristics, variables, and units of measurement (e.g., images, time series), you can streamline the description by providing details about the number of datasets, their file naming convention, and attributes once.
Preprocessed Data Folder
Content: Cleaned subsets of the PPE Latin Hypercube (LHC) data sets that are suitable for usage in a Gaussian Process Regression Machine Learning (GPR ML) Emulator. The subsets differ by user selected time range and climate variable of interest. 
File naming convention: f"{var}_{time_selection}.nc"
File attributes: The dimensions of the preprocessed LHC perturbed parameter simulations are (500,1)
Emulation Results Data

Content: Files contain our ‘pickled emulator’. In other words, the files contain character streams of the data generated by the trained GPR ML Emulator that are easy for your disk to unpack and use for more predictions and visualizations. 
File naming convention : f"{var_name}_{param_name}_{time_selection}_gpr_model.sav”
File attributes: 
```{python}
results_dict = {
	# perturbed parameter test values
    	'X_values': {},
	# climate variable predictions
    	'y_pred': {},
# climate variable predictions standard deviation
    	'y_std': {},
     	# the trained GPR ML model
    	'gpr_model': gpr_model,
     	# y_test to use to calculate R^2 later
    	'y_test': y_test,
```
Accepted Parameter Values
The txt file contains 500 accepted values for the 32 parameter features provided by the CGD – NCAR. These values were used to train the emulator for predicting parameter sensitivity and uncertainty. 
2. Relationship between files, if important:
The `preprocessed_data` files are the inputs utilized to generate the `emulated_data`. Each preprocessed file was used to train the GPR ML emulator to produce a respective output for each climate variable and parameter for the time period of interest.
Plots
FAST_accuracy_plots/
Content: 
File naming convention:f'fast_acc_plot_{var_name}_{param_name}_{time_selection}.png
emulator/
Content: 
File naming convention: f'emulator_plot_{var_name}_{param_name_upper}_{time_selection}_gpr_model.png
3. Additional related data collected that was not included in the current data package: N/A
4. Are there multiple versions of the dataset? N/A
METHODOLOGICAL INFORMATION
1. Description of methods used for collection/generation of data: <Include links or references to publications or other documentation containing experimental design or protocols used in data collection>
`Preprocessed Data`
```{python}
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ---- 	cluster reading function   	----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# modify the function if you want to pass the parameter
# pull 20 years of data
# this has cool potential to have time period subset option
# indexing the selected lists or using multiples of 500
def read_all_simulations(var, time_selection):
	'''Prepare cluster list and read to create ensemble(group of data)
	Use preprocess to select only certain dimension and a variable'''
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----  Define list of cluster lists ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# each cluster list contains 500 simulations to call
	cluster_lists = [
   	 
    	# 1995 - 2000
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.1995-02-01-00000.nc'))[1:],
    	# 2000 - 2005
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2000-02-01-00000.nc'))[1:],
    	# 2005 - 2010
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2005-02-01-00000.nc'))[1:],
     	# 2010 - 2015
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2010-02-01-00000.nc'))[1:]
	]

	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----	Prepping to Load Cluster   ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	def preprocess(ds, var):
    	'''using this function in xr.open_mfdataset as preprocess
    	ensures that when only these four things are selected
    	before the data is combined'''
    	return ds[['lat', 'lon', 'time', var]]

	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----   If-else Load Selected Time  ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# Select appropriate lists based on the time_selection

 	# Select appropriate lists based on the time_selection
	if time_selection == '2010-2015':
    	selected_files = cluster_lists[3]
   	 
    	# Read the list and load it for the notebook using combine='nested'
    	ds = xr.open_mfdataset(selected_files,
                           	combine='nested',
                           	preprocess=lambda ds: preprocess(ds, var),
                           	parallel=True,
                           	concat_dim=["ens"])
	else:
    	# up to list 1 aka 0 bc python index
    	# python end exclusive so need to go up to 4 for all
    	if time_selection == '1995-2015':
        	selected_lists = cluster_lists[:4]
     	# up to list 2 aka 1 bc python index
    	elif time_selection == '2000-2015':
        	selected_lists = cluster_lists[1:4]
     	# up to list 3 aka 2 bc python index
    	elif time_selection == '2005-2015':
        	selected_lists = cluster_lists[2:4]
    	# safety check
    	else:
        	# to ensure a user selects time range
        	raise ValueError("Uh oh, please select a time range that is currently available.")

    
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----  	Load in Cluster Data 	----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# read the list and load it for the notebook
    	ds = xr.open_mfdataset( selected_lists,
                            	combine='nested',
                           	# lambda allows us to call the predefined preprocess on the ds
                            	preprocess = lambda ds: preprocess(ds, var),
                            	parallel= True,
                            	concat_dim= ["time", "ens"])

	# we aren't going to save these files because they need to be preprocessed
	# using the wrangle and subset functions
	# better to keep these things broken up / shorter for future works updates
	# makes sense to keep if else pulling statement at the top of read_n_wrangle
	return ds

2. Methods for processing the data: <describe how the submitted data were generated from the raw or collected data>
```{python}
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ---- 	correct time-parsing bug   	----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
def fix_time(da):
    
	'''fix CESM monthly time-parsing bug'''
    
	yr0 = str(da['time.year'][0].values)
    
	da['time'] = xr.cftime_range(yr0,periods=len(da.time),freq='MS',calendar='noleap')
    
	return da

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ----  weigh dummy landarea by gridcell  ----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----  Weight Gridcells by Landarea ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

def weight_landarea_gridcells(da,landarea):

	# weigh landarea variable by mean of gridcell dimension
	return da.weighted(landarea).mean(dim = 'gridcell')

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ----   	weight var data time dim 	----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#------Weighted Averages by Time---
def yearly_weighted_average(da):
	# Get the array of number of days from the main dataset
	days_in_month = da['time.daysinmonth']

	# Multiply each month's data by corresponding days in month
	weighted_month = (days_in_month*da).groupby("time.year").sum(dim = 'time')

	# Total days in the year
	days_per_year = days_in_month.groupby("time.year").sum(dim = 'time')

	# Calculate weighted average for the year
	return weighted_month / days_per_year
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ----  	Subset Var Wrangle Funct  	----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
def wrangle_var_cluster(da):
	'''Weight gridcell dimension by landarea
	and weight time dimension by the days per each month
	over the total number of days in a year. Globally average
	the selected variable between 2005-2010
	[for now, will be time range]
	as a xr.da.'''
	# weight gridcell dim by global land area
	da_global = weight_landarea_gridcells(da, landarea)
	# weight time dim by days in month
	da_global_ann = yearly_weighted_average(da_global)
	# take global avg for variable over year dimension
	var = da_global_ann.mean(dim='year')
    
	return var
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# ---- 	cluster reading function   	----
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# modify the function if you want to pass the parameter
# pull 20 years of data
# this has cool potential to have time period subset option
# indexing the selected lists or using multiples of 500
def read_all_simulations(var, time_selection):
	'''Prepare cluster list and read to create ensemble(group of data)
	Use preprocess to select only certain dimension and a variable'''
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----  Define list of cluster lists ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# each cluster list contains 500 simulations to call
	cluster_lists = [
   	 
    	# 1995 - 2000
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.1995-02-01-00000.nc'))[1:],
    	# 2000 - 2005
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2000-02-01-00000.nc'))[1:],
    	# 2005 - 2010
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2005-02-01-00000.nc'))[1:],
     	# 2010 - 2015
    	sorted(glob.glob('/glade/campaign/cgd/tss/projects/PPE/PPEn11_LHC/transient/hist/PPEn11_transient_LHC[0][0-5][0-9][0-9].clm2.h0.2010-02-01-00000.nc'))[1:]
	]

	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----	Prepping to Load Cluster   ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	def preprocess(ds, var):
    	'''using this function in xr.open_mfdataset as preprocess
    	ensures that when only these four things are selected
    	before the data is combined'''
    	return ds[['lat', 'lon', 'time', var]]

	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----   If-else Load Selected Time  ----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# Select appropriate lists based on the time_selection

 	# Select appropriate lists based on the time_selection
	if time_selection == '2010-2015':
    	selected_files = cluster_lists[3]
   	 
    	# Read the list and load it for the notebook using combine='nested'
    	ds = xr.open_mfdataset(selected_files,
                           	combine='nested',
                           	preprocess=lambda ds: preprocess(ds, var),
                           	parallel=True,
                           	concat_dim=["ens"])
	else:
    	# up to list 1 aka 0 bc python index
    	# python end exclusive so need to go up to 4 for all
    	if time_selection == '1995-2015':
        	selected_lists = cluster_lists[:4]
     	# up to list 2 aka 1 bc python index
    	elif time_selection == '2000-2015':
        	selected_lists = cluster_lists[1:4]
     	# up to list 3 aka 2 bc python index
    	elif time_selection == '2005-2015':
        	selected_lists = cluster_lists[2:4]
    	# safety check
    	else:
        	# to ensure a user selects time range
        	raise ValueError("Uh oh, please select a time range that is currently available.")

    
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	#----  	Load in Cluster Data 	----
	#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	# read the list and load it for the notebook
    	ds = xr.open_mfdataset( selected_lists,
                            	combine='nested',
                           	# lambda allows us to call the predefined preprocess on the ds
                            	preprocess = lambda ds: preprocess(ds, var),
                            	parallel= True,
                            	concat_dim= ["time", "ens"])

	# we aren't going to save these files because they need to be preprocessed
	# using the wrangle and subset functions
	# better to keep these things broken up / shorter for future works updates
	# makes sense to keep if else pulling statement at the top of read_n_wrangle
	return ds
```

`Emulator Data`
```{python}


def train_emulator2(param, var, var_name, time_selection):
     #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     # ----         Split Data           ----
     #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    X_train, X_test, y_train, y_test = train_test_split(param,
                                                        var, 
                                                        test_size=0.2,
                                                        random_state=0)

    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # ----    Kernel Specs No Tuning    ----
    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # Kernel Specs No Tuning
    kernel = ConstantKernel(constant_value=3, constant_value_bounds=(1e-2, 1e4))  \
            * RBF(length_scale=1, length_scale_bounds=(1e-4, 1e8))

    # Using an out-of-the-box kernel for now
    gpr_model = GaussianProcessRegressor(kernel=kernel,
                                         n_restarts_optimizer=20, 
                                         random_state=99, 
                                         normalize_y=True)

    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # ----         Fit the Model        ----
    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # Fit the model to the training data
    gpr_model.fit(X_train, y_train)

    # Prepare to store results
    results_dict = {
        'X_values': {},
        'y_pred': {},
        'y_std': {},
        'r2': {},
         # save the trained GPR model
        'gpr_model': gpr_model, 
         # save y_test for R^2 later
        'y_test': y_test, 
        'X_test': X_test
       
    }

    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # ----      Iterate thru Params     ----
    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    for param_name, param_index in param_names_dict.items():
        # Create X_values for prediction linspace
        X_values = np.full((100, len(param_names_dict)), 0.5)           # r2 drops to 0.004 when removing this, but we're only using the R^2 used in fast plot
       # X_values = np.tile(X_test, 1)
        X_values[:, param_index] = np.linspace(0, 1, 100)
        # Vary only the current parameter over a linspace
        #X_values[:, param_index] = np.linspace(np.min(X_test[:, param_index]), np.max(X_test[:, param_index]))

        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # ----         Get Predictions      ----
        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # Make predictions for the current parameter
        y_pred, y_std = gpr_model.predict(X_values, return_std=True)

        
        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # ----         Collect Metrics      ----
        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        mae = mean_absolute_error(y_test, y_pred)
        rmse_emulator = np.sqrt(mean_squared_error(y_test, y_pred))
        r2_emulator = np.corrcoef(y_test, y_pred)[0, 1]**2

        # Store results in dictionaries
        results_dict['X_values'][param_name] = X_values
        results_dict['y_pred'][param_name] = y_pred
        results_dict['y_std'][param_name] = y_std
        results_dict['r2'][param_name] = r2_emulator
        
   
        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # ----      Pickle Emulation     ----
        #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # Save the predictions and overall R^2 to a file
        filename = os.path.join("emulation_results", f"{var_name}_{param_name}_{time_selection}_gpr_model.sav")

        if os.path.exists(filename):
            # Load the model from disk
            loaded_model = pickle.load(open(filename, 'rb'))
        else:
            print(f"Emulator is running for {param_name}, this may take a few moments")
            with open(filename, 'wb') as file:
                pickle.dump((X_values, y_pred, y_std, r2_emulator, param_name, var_name), file)

    return results_dict
```

3. Instrument- or software-specific information needed to interpret the data: <include full name and version of software, and any necessary packages or libraries needed to run scripts>
 - cartopy=0.22.0
  - dask=2024.1.0
  - dask-jobqueue=0.8.5
  - gpflow=2.5.2
  - hvplot=0.9.2
  - ipython=8.22.2
  - jupyter=1.0.0
  - jupyterlab=4.1.5
  - netcdf4=1.6.5
  - notebook=7.1.2
  - numpy=1.26.4
  - pandas=2.2.1
  - panel=1.3.8
  - plotly=5.19.0
  - python=3.11.7
  - regionmask=0.12.1
  - scikit-learn=1.4.1
  - scipy=1.12.0
  - seaborn=0.13.1
  - shapely=2.0.3
  - sparse=0.15.1
  - statsmodels=0.14.1
  - xarray=2024.2.0
  - xesmf=0.8.4
4. Standards and calibration information, if appropriate:
The standard of the R^2 value provided by the emulator when assessing the total LHC parameter space was above 0.65. The kernel specification below enable the GPR ML emulator to assess the covariance of the 32 parameters simulatenously to produce predictions for a given climate variable.
5. Environmental/experimental conditions: 
Kernel specification for GPR:
```{python}
    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # ----    Kernel Specs No Tuning    ----
    #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    # Kernel Specs No Tuning
    kernel = ConstantKernel(constant_value=3, constant_value_bounds=(1e-2, 1e4))  \
            * RBF(length_scale=1, length_scale_bounds=(1e-4, 1e8))
```
6. Describe any quality-assurance procedures performed on the data:
The creation of the png files was a measure to assure the GPR ML emulator predictions are performing as expected. 
9. People involved with sample collection, processing, analysis and/or submission:
DATA-SPECIFIC INFORMATION FOR:
[FILENAME] <repeat this section for each dataset, folder or file, as appropriate>
1. Number of variables:
32 perturbed parameter features for the GPR machine learning emulator to assess
10 climate variables selected for subsetting to then predict outputs for.
Dashboard allows access to over 500+ climate variables.
2. Number of cases/rows:
3. Variable List: <list variable name(s), description(s), unit(s)and value labels as appropriate for each>
4. Missing data codes: <list code/symbol and definition>
5. Specialized formats or other abbreviations used:

