# NIST-IR-MachineLearning

The aim of this project is to enable utilization of the wealth of FTIR spectra available in databases for the prediction of chemical unknown structures from FTIR spectra.  Two common approaches to this problem are to:

    1) Directly match the unknown spectra with known spectra
    2) Identify functional groups from peaks within the FTIR spectrum to hint at the full structure.
    
The general approach taken in this project will be to use known FTIR spectra to predict bits in chemical fingerprints (MACCS keys and PubChem fingerprints).  There are various types of chemical fingerprints, and they are generally methods to convert chemical structure into digital bits for chemical database searching problems (identifying similar and distinct chemicals).  In some of the fingerprints each individual bit in the fingerprint is 1 or 0 depending on whether or not a molecule has a particular molecular fragment or atom.

Prediction of the fingerprint bits from the FTIR spectra allows combination of the two strategies above.  By taking the predicted fingerprint and comparing to a database of fingerprints (even if you don't have the FTIR spectrum) it's possible to identify potential matches to the predicted fingerprint.  Additionally, based information about which bits were predicted from the spectrum would give clues as to which functional groups are present in the unknown molecule.

# NISTWebbookML.ipynb

The aim of this notebook is to determine if using FTIR spectral data to predict the presence or absence of MACCS key fingerprint bits is possible and useful. Prior work has shown some success with using machine learning to predict the presence or absence of functional groups (1,2). In this work, prediction of the bits from MACCS keys was chosen in order to see if alternate functional groups represented by the bits would be more amenable to FTIR prediction. Additionally, if successful, detection of a molecular fingerprint would allow for searching of larger chemical databases for matches outside the library by generating MACCS keys fingerprints for the library and then searching for the fingerprint of the unknown against the library of fingerprints.

A) Scrape the FTIR data and molecular information for all molecules in the NIST database
    Used prior work (1,2) and an available API (3,4)
B) Import and Clean the spectral data (same axes, units, etc.), select data for analysis
    Used prior work on jcamp import (5)
C) Learn about and Generate MACCS keys fingerprints for all molecules of interest
D) Perform Exploratory Data Analysis to Understand the Dataset
E) Evaluate Several typical Classification Models for Initial Modeling (Split the Data into Train and Test)
F) Generate Fitted Models from the Most Promising Classification Models and Predict the Test Set

References:

1) https://pubs.acs.org/doi/pdf/10.1021/acs.analchem.1c00867

2) https://github.com/Ohio-State-Allen-Lab/FTIRMachineLearning

3) https://www.linkedin.com/pulse/unofficial-nist-webbook-api-how-get-thermochemistry-data-contreras/?trk=pulse-article_more-articles_related-content-card

4) https://github.com/oscarcontrerasnavas/nist-webbook-API

5) https://github.com/nzhagen/jcamp Installed this prior to running notebook: 1 ) python -m pip install git+https://github.com/nzhagen/jcamp

My work stored here:

https://github.com/kevinpatrickkent/FTIRMachineLearning/

Here are my conclusions from working through this dataset and analysis:

A unique correct match was obtained for 44% of the tested molecules, and a match in the top 10 within the test set of 1648 molecules was obtained for 76% of molecules in the test set. The approach summarized briefly here was to obtain

1) Obtain FTIR spectra, InChI, names, cas, and various other meta data from NIST
2) Clean the data to get spectra in the same state (gas), y-units (transmittince), and x-units (wavenumber)
3) Genearte MACCS keys from the InChI for each molecule with RDKit (166 bits - some removed in cleaning)
4) Normalize the the transmittance spectra with Min Max normalization and reduce dimensionality with PCA (sklearn)
5) Initial modeling with K-Nearest Neighbors, Suppor Vector Classifier, Logisitic Regression, Decision Tree, Random Forest, Adaboost, and Naive-Bayes scored with Jaccard/Tanimoto Index
6) Modeled expected molecular Jaccard Index expected from obtaining particular Jaccard Index Values for MACCS Keys bits
7) Optimized the two most promising models (KNN and SVC) with a scoring metric of getting as many as possible bit Jaccard index values above 0.8 as possible.
8) Chose SVC to move forward (optimized C value of 3) --> Tested different Jaccard value index cutoffs for removing MACCS keys bits from the analysis (tested this on the final test set).
9) Selected a cutoff of 0.3 based on the trade-off between obtaining a higher Jaccard Index and maintaining bits to keep molecules unique from one another.
10) Generated the final SVC model.

Going back through on this post analysis, there is a clear mismatch between the obtained molecular jaccard index (greater thatn 50% above 0.5) and the cutoff for the bits (Jaccard Index of 0.3). If you look back at 6 where I modeled the expected molecular Jaccard index - I did not take into account the distribution of Jaccard index. I simply modeled the Jaccard index as a monolithic value. Perhaps the distribution of Jaccard index for the features would aid future modeling of expected molecular Jaccard index. In fact, some additional work in this area could aid in determining how useful adding more bits would be to this process.

In the future it would be interested to generate these fingerprints for a larger database of values (PubChem for instance) and see how well the fingerprints generated from these spectra allow for identification of a molecule fro PubChem. My guess would be that the lower jaccard index matches would not fare well as the database size increases.

For the preprocessing of the data with PCA, I didn't put any time or analysis into evaluation or optimization of that step. Removing the processing step, using PLS, and peak fitting could all be tried in order to improve the capture of spectral features.

I selected the MACCS keys - but there are many fingerprints available for digitizing chemical structure. I intially started with RDKit fingerprints, but it was not clear to me from the documentation that each bit would come from a unique feature. So, I dropped the RDKit fingerprint in favor of MACCS keys. However, perhaps other fingerprints could be used to generate several potential FTIR fragments to be used in this type analysis. I think this area may lead to the most improvement in future analysis.

The best next step would likely be to expand the matching to a larger database and get a benchmark of where this approach stands right now before making additional improvements. For now, I will be moving on to other projects for a while and hopefully come back at some point to visit this project at a later date.
