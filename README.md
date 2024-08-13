# SBG-TIR OTTER Level 2 Surface Temperature & Emissivity (L2 LSTE) Data Product

This repository will contain the Surface Biology and Geology Thermal Infrared (SBG-TIR) OTTER level 2 land-surface temperature and emissivity (L2 LSTE) data product algorithm.

The SBG collection 1 level 2 surface temperature data product is being developed based on the [ECOsystem Spaceborne Thermal Radiometer Experiment on Space Station (ECOSTRESS) collection 3 level 2 surface temperature data product](https://github.com/ECOSTRESS-Collection-3/ECOv003-L2-LSTE).

[Glynn C. Hulley](https://github.com/ghulley) (he/him)<br>
[glynn.hulley@jpl.nasa.gov](mailto:glynn.hulley@jpl.nasa.gov)<br>
Lead developer and designer<br>
NASA Jet Propulsion Laboratory 329G

##  1. Introduction

This document outlines the theory and methodology for generating the OTTER Level-2 (L2) land surface temperature and emissivity (LST&E) products. The LST product is derived from the six TIR spectral bands between 8 and 12.5 µm, while the emissivity is retrieved for all 6 TIR and 2 MIR bands. The LST&E products are retrieved from the surface spectral radiance that is obtained by atmospherically correcting the at-sensor spectral radiance. Knowledge of the surface emissivity is critical for accurately recovering the surface temperature, a key climate variable in many scientific studies from climatology to hydrology, modeling the greenhouse effect, drought monitoring, and land surface models (Anderson et al. 2007; French et al. 2005; Jin and Dickinson 2010).

In addition to surface energy balance, LST&E products are essential for a wide range of other Earth system studies. For example, emissivity spectral signatures are important for geologic studies and mineral mapping studies (Hook et al. 2005; Vaughan et al. 2005). This is because emissivity features in the TIR region are unique for many different types of materials that make up the Earth's surface, for example, quartz, which is ubiquitous in most of the arid regions of the world. Emissivities are also used for land use and land cover change mapping since vegetation fractions can often be inferred if the background soil is observable (French et al. 2008). 

Maximum radiometric emission for the typical range of Earth surface temperatures, excluding fires and volcanoes, is found in two infrared spectral "window" regions: the midwave infrared (3.5–5 µm) and the thermal infrared (8–13 µm). The radiation emitted in these windows for a given wavelength is a function of both temperature and emissivity. Determining the separate contribution from each component in a radiometric measurement is an ill-posed problem since there will always be more unknowns—N emissivities and a single temperature—than the number of measurements, N, available. For SBG, we will be solving for one temperature and eight emissivities. Therefore, an additional constraint is needed, independent of the data. There have been numerous theories and approaches over the past two decades to solve for this extra degree of freedom. For example, the ASTER Temperature Emissivity Working Group (TEWG) analyzed ten different algorithms for solving the problem (Gillespie et al. 1999). Most of these relied on a radiative transfer model to correct at-sensor radiance to surface radiance and an emissivity model to separate temperature and emissivity. Other approaches include the split-window (SW) algorithm, which extends the SST SW approach to land surfaces, assuming that land emissivities in the window region (10.5–12 µm) are stable and well known. However, this assumption leads to unreasonably large errors over barren regions where emissivities have large variations both spatially and spectrally. The ASTER TEWG finally decided on a hybrid algorithm, termed the temperature emissivity separation (TES) algorithm, which capitalizes on the strengths of previous algorithms with additional features (Gillespie et al. 1998). 

The remainder of the document will discuss the SBG instrument characteristics, provide a background on TIR remote sensing, give a full description and background on the atmospheric correction and the TES algorithm, provide quality assessment, discuss numerical simulation studies and, finally, outline a validation plan.

## 2. Data Products

### 2.1. Metadata

SBG standards incorporate additional metadata that describe each HDF5 Dataset within the HDF5 file. Each of these metadata elements appear in an HDF5 Attribute that is directly associated with the HDF5 Dataset. Wherever possible, these HDF5 Attributes employ names that conform to the Climate and Forecast (CF) conventions. Table 2-3 lists the CF names for the HDF5 Attributes that SBG products typically employ.

Each SBG product bundle contains at least two sets of product metadata:
-   ProductMetadata
-   StandardMetadata

#### 2.1.1. Standard Metadata
Information on the `StandardMetadata` is included on the [SBG-TIR github landing page](https://github.com/sbg-tir)

#### 2.1.2. Product Metadata

| **CF Compliant Attribute Name** | **Description** | **Required?** | 
| --- | --- | --- |
| Units | Units of measure.  Appendix A lists applicable units for various data elements in this product. | Yes |
| valid_max | The largest valid value for any element in the Dataset.  The data type in valid_max matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding valid_max will also be float32. | No |
| valid_min | The smallest valid value for any element in the Dataset.  The data type in valid_min matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding valid_min will also be float32. | No |
| _FillValue | Specification of the value that will appear in the Dataset when an element is missing or undefined.  The data type of _FillValue matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding _FillValue will also be float32. Datasets that do not have a fill value will omit this attribute. | No |
| long_name | A descriptive name that clearly describes the content of the associated Dataset. | Yes |

*Table 1. SBG Specific Local Attributes*

| **Group** | **Type** | **Size** | **Example** | 
| --- | --- | --- | --- |
| QAPercentCloudCover | Int | 4 | 80 | 
| CloudMeanTemperature | LongFloat | 8 | 231 |
| CloudMaxTemperature | LongFloat | 8 | 275 |
| CloudMinTemperature | LongFloat | 8 | 221 |
| CloudSDevTemperature | LongFloat | 8 | 0.45 |
| QAFractionGoodQuality | Int | 4 | 0.7 |
| LSTGoodAvg | LongFloat | 8 | 285.4 |
| Emis3GoodAvg | LongFloat | 8 | 0.95 |
| Emis4GoodAvg | LongFloat | 8 | 0.95 |
| Emis5GoodAvg | LongFloat | 8 | 0.95 |
| Emis6GoodAvg | LongFloat | 8 | 0.95 |
| Emis7GoodAvg | LongFloat | 8 | 0.95 |
| Emis8GoodAvg | LongFloat | 8 | 0.95 |
| Emis9GoodAvg | LongFloat | 8 | 0.95 |
| Emis10GoodAvg | LongFloat | 8 | 0.95 |
| AncillaryGEOS5 | String | 255 |  |
| BandSpecification | Float32 | μm |  |

*Table 2. L2 LSTE Product Metadata Definitions*

| **Name** | **Type** | **Size** | **Example** | 
| --- | --- | --- | --- |
| QAPercentCloudCover | Int | 4 | 80 |
| CloudMeanTemperature | LongFloat | 8 | 231 |
| CloudMaxTemperature | LongFloat | 8 | 275 |
| CloudMinTemperature | LongFloat | 8 | 221 |
| CloudSDevTemperature | LongFloat | 8 | 0.45 |

*Table 3. L2 CLOUD Product Metadata Definitions*

### 2.2. L2 LSTE Products

| **SDS** | **Long Name** | **Data Type** | **Units** | **Valid Range** | **Fill Value** | **Scale Factor** | **Offset** | 
| --- | --- | --- | --- | --- | --- | --- | --- |
| LST | Land Surface Temperature | uint16 | K | 7500-65535 | 0 | 0.02 | 0 |
| QC | Quality control for LST and emissivity | uint16 | N/A | 0-65535 | N/A | N/A | N/A |
| Emis3 | Band 3 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis4 | Band 4 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis5 | Band 5 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis6 | Band 6 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis7 | Band 7 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis8 | Band 8 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis9 | Band 9 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| Emis10 | Band 10 emissivity  | uint8 | N/A | 1-255 | 0 | 0.002 | 0.49 |
| LST_Err | Land Surface Temperature error  | uint8 | K | 1-255 | 0 | 0.04 | 0 |
| Emis3_Err | Band 3 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis4_Err | Band 4 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis5_Err | Band 5 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis6_Err | Band 6 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis7_Err | Band 7 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis8_Err | Band 8 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis9_Err | Band 9 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| Emis10_Err | Band 10 emissivity error  | uint16 | N/A | 0-65535 | 0 | 0.0001 | 0 |
| EmisWB | Wideband emissivity  | uint8 | N/A | 0-255 | 0 | 0.002 | 0.49 |
| PWV | Precipitable Water Vapor  | uint16 | cm | 0-65535 | N/A | 0.0001 | 0 |
| water_mask | Land/water mask | uint8 | 1=water, 0=land | 0-1 | 255 | 1 | 0 |
| cloud | Land/water mask | uint8 | 1=cloud, 0=clear | 0-1 | 255 | 1 | 0 |
| height | Ground elevation | uint16 | meters| -1000-10000 | -32768 | 1 | 0 |
| range | Satellite to pixel range | uint16 | meters| 0-32767 | -32768 | 100 | 800000 |
| view_zenith | Sensor zenith angle | uint16 | degrees |  0-18000 | -32768 | 0.01 | 0 |

*Table 3-4: Product Data Definitions for the L2 LSTE Product*

| **Bits** | **Long Name** | **Description** | 
| --- | --- | --- | 
| 1&0 | Mandatory QA flags | 00 = Pixel produced, best quality
01 = Pixel produced, nominal quality. Either one or more of the following conditions are met: 
1.	emissivity in both bands 9 and 10 < 0.95,  i.e. possible cloud contamination
2.	low transmissivity due to high water vapor loading (<0.4), check PWV values and error estimates
Recommend more detailed analysis of   other QC information
10 = Pixel produced, but cloud detected
11 = Pixel not produced due to missing/bad data, user should check Data quality flag bits | 
|   3 & 2 | Data quality flag | 00 = Good quality L1B data
01 = not set
10 = not set
11 = Missing/bad L1B data | 
| 5 & 4 | Cloud/Ocean Flag | Not set. Please check SBG GEO and CLOUD products for this information. | 
| 7 & 6 | Iterations | 00 = Slow convergence
01 = Nominal
10 = Nominal
11 = Fast | 
| 9 & 8 | Atmospheric Opacity | 00 = >=3 (Warm, humid air; or cold land)
01 = 0.2 - 0.3 (Nominal value)
10 = 0.1 - 0.2 (Nominal value)
11 = <0.1 (Dry, or high altitude pixel)
 | 
| 11 & 10 | MMD | 00 = > 0.15 (Most silicate rocks)
01 = 0.1 - 0.15 (Rocks, sand, some soils)
10 = 0.03 - 0.1 (Mostly soils, mixed pixel)
11 = <0.03 (Vegetation, snow, water, ice)
 | 
| 13 & 12 | Emissivity accuracy  | 00 = >0.02 (Poor performance)
01 = 0.015 - 0.02 (Marginal performance)
10 = 0.01 - 0.015 (Good performance)
11 = <0.01 (Excellent performance) | 
| 15 & 14 | LST accuracy | 00 = >2 K (Poor performance)
01 = 1.5 - 2 K (Marginal performance)
10 = 1 - 1.5 K (Good performance)
11 = <1 K (Excellent performance) | 

*Table 3-5: Bit flags defined in the QC SCS*

### 2.3. L2 CLOUD data

| **SDS** | **Long Name** | **Data Type** | **Units** | **Valid Range** | **Fill Value** | **Scale Factor** | **Offset** | 
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cloud_confidence | Brightness temperature LUT test | uint8 | 3=confident cloudy, 2=probably cloudy, 1=probably clear, 0=confident clear | 0-1 | 255 | 1 | 0 |
| Cloud_final | Final cloud mask | uint8 | 1=cloud, 0=clear | 0-1 | 255 | 1 | 0 |

*Table 3-6: Product Data Definitions for the L2 Cloud Product*

| **Name** | **Type** | **Size** | **Example** | 
| --- | --- | --- | --- |
| QAPercentCloudCover | Int | 4 | 80 |
| CloudMeanTemperature | LongFloat | 8 | 231 |
| CloudMaxTemperature | LongFloat | 8 | 275 |
| CloudMinTemperature | LongFloat | 8 | 221 |
| CloudSDevTemperature | LongFloat | 8 | 0.45 |

*Table 3-7: Metadata Definitions for the L2 Cloud Product*

### 2.4. Low latency product

A low latency (\< 24 hour) product will be created in addition to the standard product. It may or may not be archived. The product contents will be the same as the standard product, although the method to obtain it will be different (see the relevant ATBD).

## 3. Theory and Methodology

The at-sensor measured radiance in the infrared region (4–15 µm, MIR: 3-5 µm, TIR: 8-15 µm) consists of a combination of different terms from surface emission, solar reflection, and atmospheric emission and attenuation. The Earth-emitted radiance is a function of temperature and emissivity and gets attenuated by the atmosphere on its path to the satellite. The emissivity of an isothermal, homogeneous emitter is defined as the ratio of the actual emitted radiance to the radiance emitted from a black body at the same thermodynamic temperature (Norman and Becker 1995),  ϵ_λ= R_λ/B_λ. The emissivity is an intrinsic property of the Earth’s surface and is an independent measurement of the surface temperature, which varies with irradiance and local atmospheric conditions. The emissivity of most natural Earth surfaces for the TIR wavelength ranges between 8 and 12 μm and, for a sensor with spatial scales <100 m, varies from ~0.7 to close to 1.0. Narrowband emissivities less than 0.85 are typical for most desert and semi-arid areas due to the strong quartz absorption feature (reststrahlen band) between the 8- and 9.5-μm range, whereas the emissivity of vegetation, water, and ice cover are generally greater than 0.95 and spectrally flat in the 3–15-μm range except for dry and senesced vegetation that can have emissivities range from 0.9-0.95 in the longer wavelengths above 10 μm.

TES is applied to the land-leaving TIR radiances that are estimated by atmospherically correcting the at-sensor radiance on a pixel-by-pixel basis using a radiative transfer model. TES uses an empirical relationship to predict the minimum emissivity that would be observed from a given spectral contrast, or minimum-maximum difference (MMD) (Kealy and Hook 1993; Matsunaga 1994). The empirical relationship is referred to as the calibration curve and is derived from a subset of spectra in the ASTER spectral library (Baldridge et al. 2009). A new calibration curve, applicable to SBG TIR bands, will be computed using the latest ECOSTRESS spectral library v2 (Meerdink et al. 2019), in addition to spectra from 9 pseudo-invariant sand dune sites located in the US Southwest (Hulley et al. 2009a). TES has been shown to accurately recover temperatures within 1 K and emissivities within 0.015 for a wide range of surfaces and is a well established physical algorithm that produces seamless images with no artificial discontinuities such as might be seen in a land classification type algorithm (Gillespie et al. 1998).

## 4. Uncertainty Analysis 

NASA has identified a major need to develop long-term, consistent products valid across multiple missions, with well-defined uncertainty statistics addressing specific Earth-science questions. These products are termed Earth System Data Records (ESDRs), and LST&E has been identified as an important ESDR. Currently a lack of understanding of LST&E uncertainties limits their usefulness in land surface and climate models. In this section we present results from the Temperature Emissivity Uncertainty Simulator (TEUSim) that has been developed to quantify and model uncertainties for a variety of TIR sensors and LST algorithms (Hulley et al. 2012b). Using the simulator, uncertainties were estimated for the L2 products of SBG using a 6-band TES approach. These uncertainties will be parameterized according to view angle and estimated total column water vapor for eventual application to real-time SBG L2 data on a pixel by pixel basis.

## 5. Cal/Val

The OTTER L2 LST product will be validated with a combination of Temperature-based (Coll et al. 2005; Hook et al. 2004) and Radiance-based methods (Hulley and Hook 2012; Wan and Li 2008) using a global set of validation sites. The L2 emissivity product will be validated using a combination of lab-measured samples collected at various sand dune sites, and with the ASTER Global Emissivity Database (ASTER GED)  (Hulley and Hook 2009b). 


#### Acknowledgements 

The research was carried out at the Jet Propulsion Laboratory, California Institute of Technology, under a contract with the National Aeronautics and Space Administration.

#### References

[]{#_ENREF_1 .anchor}Anderson, M.C., Norman, J.M., Mecikalski, J.R., Otkin, J.A., & Kustas, W.P. (2007). A climatological study of evapotranspiration and moisture stress across the continental United States based on thermal remote sensing: 2. Surface moisture climatology. *Journal of Geophysical Research-Atmospheres, 112*

[]{#_ENREF_2 .anchor}Augustine, J.A., DeLuisi, J.J., & Long, C.N. (2000). SURFRAD - A national surface radiation budget network for atmospheric research. *Bulletin of the American Meteorological Society, 81*, 2341-2357

[]{#_ENREF_3 .anchor}Baldridge, A.M., Hook, S.J., Grove, C.I., & Rivera, G. (2009). The ASTER Spectral Library Version 2.0. *Remote Sensing of Environment, 114*, 711-715

[]{#_ENREF_4 .anchor}Bannari, A., Omari, K., Teillet, R.A., & Fedosejevs, G. (2005). Potential of getis statistics to characterize the radiometric uniformity and stability of test sites used for the calibration of earth observation sensors. *Ieee Transactions on Geoscience and Remote Sensing, 43*, 2918-2926

[]{#_ENREF_5 .anchor}Barducci, A., & Pippi, I. (1996). Temperature and emissivity retrieval from remotely sensed images using the \'\'grey body emissivity\'\' method. *Ieee Transactions on Geoscience and Remote Sensing, 34*, 681-695

[]{#_ENREF_6 .anchor}Barton, I.J., Minnett, P.J., Maillet, K.A., Donlon, C.J., Hook, S.J., Jessup, A.T., & Nightingale, T.J. (2004). The Miami2001 infrared radiometer calibration and intercomparison. part II: Shipboard results. *Journal of Atmospheric and Oceanic Technology, 21*, 268-283

[]{#_ENREF_7 .anchor}Barton, I.J., Zavody, A.M., Obrien, D.M., Cutten, D.R., Saunders, R.W., & Llewellyn-Jones, D.T. (1989). Theoretical Algorithms for Satellite-Derived Sea-Surface Temperatures. *Journal of Geophysical Research-Atmospheres, 94*, 3365-3375

[]{#_ENREF_8 .anchor}Bauer, P., Moreau, E., Chevallier, F., & O\'Keeffe, U. (2006). Multiple-scattering microwave radiative transfer for data assimilation applications. *Quarterly Journal of the Royal Meteorological Society, 132*, 1259-1281

[]{#_ENREF_9 .anchor}Becker, F., & Li, Z.L. (1990). Temperature-Independent Spectral Indexes in Thermal Infrared Bands. *Remote Sensing of Environment, 32*, 17-33

[]{#_ENREF_10 .anchor}Berk, A., Anderson, G.P., Acharya, P.K., Bernstein, L.S., Muratov, L., Lee, J., Fox, M., Adler-Golden, S.M., Chetwynd, J.H., Hoke, M.L., Lockwood, R.B., Gardner, J.A., Cooley, T.W., Borel, C.C., & Lewis, P.E. (2005). MODTRAN^TM^ 5, A Reformulated Atmospheric Band Model with Auxiliary Species and Practical Multiple Scattering Options: Update. In S.S. Sylvia, & P.E. Lewis (Eds.), *in Proc SPIE, Algorithms and Technologies for Multispectral, Hyperspectral, and Ultraspectral Imagery XI* (pp. 662-667). Bellingham, WA: Proceedings of SPIE

[]{#_ENREF_11 .anchor}Borbas, E., Seemann, S.W., Huang, H.L., Li, J., & Menzel, W.P. (2005). Global profile training database for satellite regression retrievals with estimates of skin temperature and emissivity. In, *Proc. of the Int. ATOVS Study Conference-XIV*

[]{#_ENREF_12 .anchor}Bosilovich, M.G., Chen, J.Y., Robertson, F.R., & Adler, R.F. (2008). Evaluation of global precipitation in reanalyses. *Journal of Applied Meteorology and Climatology, 47*, 2279-2299

[]{#_ENREF_13 .anchor}Brown, O., & Minnett, P. (1999). MODIS infrared sea surface temperature algorithm. *Algorithm Theoretical Basis Document Version 2*, Univ. of Miami, Miami, Fla.

[]{#_ENREF_14 .anchor}Chevallier, F. (2000). Sampled database of 60-level atmospheric profiles from the ECMWF analyses. In

[]{#_ENREF_15 .anchor}Coll, C., & Caselles, V. (1997). A split-window algorithm for land surface temperature from advanced very high resolution radiometer data: Validation and algorithm comparison. *Journal of Geophysical Research-Atmospheres, 102*, 16697-16713

[]{#_ENREF_16 .anchor}Coll, C., Caselles, V., Galve, J.M., Valor, E., Niclos, R., Sanchez, J.M., & Rivas, R. (2005). Ground measurements for the validation of land surface temperatures derived from AATSR and MODIS data. *Remote Sensing of Environment, 97*, 288-300

[]{#_ENREF_17 .anchor}Coll, C., Wan, Z.M., & Galve, J.M. (2009a). Temperature-based and radiance-based validations of the V5 MODIS land surface temperature product. *Journal of Geophysical Research-Atmospheres, 114*, D20102, doi:20110.21029/22009JD012038

[]{#_ENREF_18 .anchor}Coll, C., Wan, Z.M., & Galve, J.M. (2009b). Temperature-based and radiance-based validations of the V5 MODIS land surface temperature product. *Journal of Geophysical Research-Atmospheres, 114*, -

[]{#_ENREF_19 .anchor}Cook, M. (2014). Atmospheric Compensation for a Landsat Land Surface Temperature Product. In, *Chester F. Carlson Center for Imaging Science*. Rochester: Rochester Institute of Technology

[]{#_ENREF_20 .anchor}Cosnefroy, H.N., Leroy, M., & Briottet, X. (1996). Selection and characterization of Saharan and Arabian desert sites for the calibration of optical satellite sensors. *Remote Sensing of Environment, 58*, 101-114

[]{#_ENREF_21 .anchor}Coudert, B., Ottle, C., Boudevillain, B., Demarty, J., & Guillevic, P. (2006). Contribution of thermal infrared remote sensing data in multiobjective calibration of a dual-source SVAT model. *Journal of Hydrometeorology, 7*, 404-420

[]{#_ENREF_22 .anchor}de Vries, C., Danaher, T., Denham, R., Scarth, P., & Phinn, S. (2007). An operational radiometric calibration procedure for the Landsat sensors based on pseudo-invariant target sites. *Remote Sensing of Environment, 107*, 414-429

[]{#_ENREF_23 .anchor}Deschamps, P.Y., & Phulpin, T. (1980). Atmospheric Correction of Infrared Measurements of Sea-Surface Temperature Using Channels at 3.7, 11 and 12 Mu-M. *Boundary-Layer Meteorology, 18*, 131-143

[]{#_ENREF_24 .anchor}Eyre, J.R., & Woolf, H.M. (1988). Transmittance of Atmospheric Gases in the Microwave Region - a Fast Model. *Applied Optics, 27*, 3244-3249

[]{#_ENREF_25 .anchor}French, A.N., Jacob, F., Anderson, M.C., Kustas, W.P., Timmermans, W., Gieske, A., Su, Z., Su, H., McCabe, M.F., Li, F., Prueger, J., & Brunsell, N. (2005). Surface energy fluxes with the Advanced Spaceborne Thermal Emission and Reflection radiometer (ASTER) at the Iowa 2002 SMACEX site (USA) (vol 99, pg 55, 2005). *Remote Sensing of Environment, 99*, 471-471

[]{#_ENREF_26 .anchor}French, A.N., Schmugge, T.J., Ritchie, J.C., Hsu, A., Jacob, F., & Ogawa, K. (2008). Detecting land cover change at the Jornada Experimental Range, New Mexico with ASTER emissivities. *Remote Sensing of Environment, 112*, 1730-1748

[]{#_ENREF_27 .anchor}Galve, J.A., Coll, C., Caselles, V., & Valor, E. (2008). An atmospheric radiosounding database for generating land surface temperature algorithms. *IEEE Transactions on Geoscience and Remote Sensing, 46*, 1547-1557

[]{#_ENREF_28 .anchor}Gillespie, A., Rokugawa, S., Hook, S., Matsunaga, T., & Kahle, A.B. (1999). Temperature/Emissivity Separation Algorithm Theoretical Basis Document, Version 2.4, ASTER TES ATBD, NASA Contract NAS5-31372, 31322 March, 31999

[]{#_ENREF_29 .anchor}Gillespie, A., Rokugawa, S., Matsunaga, T., Cothern, J.S., Hook, S., & Kahle, A.B. (1998). A temperature and emissivity separation algorithm for Advanced Spaceborne Thermal Emission and Reflection Radiometer (ASTER) images. *Ieee Transactions on Geoscience and Remote Sensing, 36*, 1113-1126

[]{#_ENREF_30 .anchor}Gottsche, F.M., & Hulley, G.C. (2012a). Validation of six satellite-retrieved land surface emissivity products over two land cover types in a hyper-arid region. *Remote Sensing of Environment, 124*, 149-158

[]{#_ENREF_31 .anchor}Gottsche, F.M., & Hulley, G.C. (2012b). Validation of six satellite-retrieved land surface emissivity products over two land cover types in a hyper-arid region. *Remote Sensing of Environment, 124*, 149-158

[]{#_ENREF_32 .anchor}Guillevic, P., Privette, J.L., Coudert, B., Palecki, M.A., Demarty, J., Ottle, C., & Augustine, J.A. (2012). Land Surface Temperature product validation using NOAA\'s surface climate observation networks---Scaling methodology for the Visible Infrared Imager Radiometer Suite (VIIRS). *Remote Sensing of Environment, 124*, 282-298

[]{#_ENREF_33 .anchor}Gustafson, W.T., Gillespie, A.R., & Yamada, G.J. (2006). Revisions to the ASTER temperature/emissivity separation algorithm. In, *2nd International Symposium on Recent Advances in Quantitative Remote Sensing*. Torrent (Valencia), Spain

[]{#_ENREF_34 .anchor}Hook, S.J., Chander, G., Barsi, J.A., Alley, R.E., Abtahi, A., Palluconi, F.D., Markham, B.L., Richards, R.C., Schladow, S.G., & Helder, D.L. (2004). In-flight validation and recovery of water surface temperature with Landsat-5 thermal infrared data using an automated high-altitude lake validation site at Lake Tahoe. *Ieee Transactions on Geoscience and Remote Sensing, 42*, 2767-2776

[]{#_ENREF_35 .anchor}Hook, S.J., Dmochowski, J.E., Howard, K.A., Rowan, L.C., Karlstrom, K.E., & Stock, J.M. (2005). Mapping variations in weight percent silica measured from multispectral thermal infrared imagery - Examples from the Hiller Mountains, Nevada, USA and Tres Virgenes-La Reforma, Baja California Sur, Mexico. *Remote Sensing of Environment, 95*, 273-289

[]{#_ENREF_36 .anchor}Hook, S.J., Gabell, A.R., Green, A.A., & Kealy, P.S. (1992). A Comparison of Techniques for Extracting Emissivity Information from Thermal Infrared Data for Geologic Studies. *Remote Sensing of Environment, 42*, 123-135

[]{#_ENREF_37 .anchor}Hook, S.J., Prata, F.J., Alley, R.E., Abtahi, A., Richards, R.C., Schladow, S.G., & Palmarsson, S.O. (2003). Retrieval of lake bulk and skin temperatures using Along-Track Scanning Radiometer (ATSR-2) data: A case study using Lake Tahoe, California. *Journal of Atmospheric and Oceanic Technology, 20*, 534-548

[]{#_ENREF_38 .anchor}Hook, S.J., Vaughan, R.G., Tonooka, H., & Schladow, S.G. (2007). Absolute radiometric in-flight validation of mid infrared and thermal infrared data from ASTER and MODIS on the terra spacecraft using the Lake Tahoe, CA/NV, USA, automated validation site. *Ieee Transactions on Geoscience and Remote Sensing, 45*, 1798-1807

[]{#_ENREF_39 .anchor}Hulley, G., Hook, S., & Hughes, C. (2012a). MODIS MOD21 Land Surface Temperature and Emissivity Algorithm Theoretical Basis Document. In: Jet Propulsion Laboratory, California Institute of Technology, JPL Publication 12-17, August, 2012

[]{#_ENREF_40 .anchor}Hulley, G.C., Gottsche, F.M., Rivera, G., Hook, S.J., Freepartner, R.J., Martin, M.A., Cawse-Nicholson, K., & Johnson, W.R. (2022). Validation and Quality Assessment of the ECOSTRESS Level-2 Land Surface Temperature and Emissivity Product. *Ieee Transactions on Geoscience and Remote Sensing, 60*

[]{#_ENREF_41 .anchor}Hulley, G.C., & Hook, S.J. (2009a). Intercomparison of Versions 4, 4.1 and 5 of the MODIS Land Surface Temperature and Emissivity Products and Validation with Laboratory Measurements of Sand Samples from the Namib Desert, Namibia. *Remote Sensing of Environment, 113*, 1313-1318

[]{#_ENREF_42 .anchor}Hulley, G.C., & Hook, S.J. (2009b). The North American ASTER Land Surface Emissivity Database (NAALSED) Version 2.0. *Remote Sensing of Environment, 113*, 1967-1975

[]{#_ENREF_43 .anchor}Hulley, G.C., & Hook, S.J. (2011). Generating Consistent Land Surface Temperature and Emissivity Products Between ASTER and MODIS Data for Earth Science Research. *Ieee Transactions on Geoscience and Remote Sensing, 49*, 1304-1315

[]{#_ENREF_44 .anchor}Hulley, G.C., & Hook, S.J. (2012). A radiance-based method for estimating uncertainties in the Atmospheric Infrared Sounder (AIRS) land surface temperature product. *Journal of Geophysical Research-Atmospheres, 117*

[]{#_ENREF_45 .anchor}Hulley, G.C., Hook, S.J., & Baldridge, A.M. (2008). ASTER land surface emissivity database of California and Nevada. *Geophysical Research Letters, 35*, L13401, doi: 13410.11029/12008gl034507

[]{#_ENREF_46 .anchor}Hulley, G.C., Hook, S.J., & Baldridge, A.M. (2009a). Validation of the North American ASTER Land Surface Emissivity Database (NAALSED) Version 2.0 using Pseudo-Invariant Sand Dune Sites. *Remote Sensing of Environment, 113*, 2224-2233

[]{#_ENREF_47 .anchor}Hulley, G.C., Hook, S.J., Manning, E., Lee, S.Y., & Fetzer, E.J. (2009b). Validation of the Atmospheric Infrared Sounder (AIRS) Version 5 (v5) Land Surface Emissivity Product over the Namib and Kalahari Deserts. *Journal of Geophysical Research Atmospheres, 114*, D19104

[]{#_ENREF_48 .anchor}Hulley, G.C., Hughes, T., & Hook, S.J. (2012b). Quantifying Uncertainties in Land Surface Temperature (LST) and Emissivity Retrievals from ASTER and MODIS Thermal Infrared Data. *Journal of Geophysical Research Atmospheres*, in press.

[]{#_ENREF_49 .anchor}Jimenez-Munoz, J.C., & Sobrino, J.A. (2010). A Single-Channel Algorithm for Land-Surface Temperature Retrieval From ASTER Data. *Ieee Geoscience and Remote Sensing Letters, 7*, 176-179

[]{#_ENREF_50 .anchor}Jin, M.L., & Dickinson, R.E. (2010). Land surface skin temperature climatology: benefitting from the strengths of satellite observations. *Environmental Research Letters, 5*, -

[]{#_ENREF_51 .anchor}Justice, C., & Townshend, J. (2002). Special issue on the moderate resolution imaging spectroradiometer (MODIS): a new generation of land surface monitoring. *Remote Sensing of Environment, 83*, 1-2

[]{#_ENREF_52 .anchor}Kalnay, E., Kanamitsu, M., & Baker, W.E. (1990). Global Numerical Weather Prediction at the National-Meteorological-Center. *Bulletin of the American Meteorological Society, 71*, 1410-1428

[]{#_ENREF_53 .anchor}Kealy, M.J., Montgomery, M., & Dovidio, J.F. (1990). Reliability and Predictive-Validity of Contingent Values - Does the Nature of the Good Matter. *Journal of Environmental Economics and Management, 19*, 244-263

[]{#_ENREF_54 .anchor}Kealy, P.S., & Hook, S. (1993). Separating temperature & emissivity in thermal infrared multispectral scanner data: Implication for recovering land surface temperatures. *Ieee Transactions on Geoscience and Remote Sensing, 31*, 1155-1164

[]{#_ENREF_55 .anchor}Kneizys, F.X., Abreu, L.W., Anderson, G.P., Chetwynd, J.H., Shettle, E.P., Berk, A., Bernstein, L.S., Robertson, D.C., Acharya, P.K., Rothman, L.A., Selby, J.E.A., Gallery, W.O., & Clough, S.A. (1996). The MODTRAN 2/3 Report & LOWTRAN 7 Model, F19628-91-C-0132. In, *Phillips Lab*. Hanscom AFB, MA

[]{#_ENREF_56 .anchor}Li, F.Q., Jackson, T.J., Kustas, W.P., Schmugge, T.J., French, A.N., Cosh, M.H., & Bindlish, R. (2004). Deriving land surface temperature from Landsat 5 and 7 during SMEX02/SMACEX. *Remote Sensing of Environment, 92*, 521-534

[]{#_ENREF_57 .anchor}Lucchesi, R. (2017). File Specification for GEOS-5 FP. GMAO Office Note No. 4 (Version 1.1), 61 pp, available from <http://gmao.gsfc.nasa.gov/pubs/office_notes>. In

[]{#_ENREF_58 .anchor}Lyon, R. (1965). Analysis of ROcks by Spectral INfrared Emission (8 to 25 microns). *Economic Geology and the Bulletin of the Society of Economic Geologists, 60*, 715-736

[]{#_ENREF_59 .anchor}Masuda, K., Takashima, T., & Takayama, Y. (1988). Emissivity of Pure and Sea Waters for the Model Sea-Surface in the Infrared Window Regions. *Remote Sensing of Environment, 24*, 313-329

[]{#_ENREF_60 .anchor}Matricardi, M. (2008). The generation of RTTOV regression coefficients for IASI and AIRS using a new profile training set and a new line-by-line database. In: ECMWF Research Dept. Tech. Memo.

[]{#_ENREF_61 .anchor}Matricardi, M. (2009). Technical Note: An assessment of the accuracy of the RTTOV fast radiative transfer model using IASI data. *Atmospheric Chemistry and Physics, 9*, 6899-6913

[]{#_ENREF_62 .anchor}Matricardi, M., Chevallier, F., Kelly, G., & Thepaut, J.N. (2004). An improved general fast radiative transfer model for the assimilation of radiance observations. *Quarterly Journal of the Royal Meteorological Society, 130*, 153-173

[]{#_ENREF_63 .anchor}Matricardi, M., Chevallier, F., & Tjemkes, S.A. (2001). An improved general fast radiative transfer model for the assimilation of radiance observations. In: European Centre for Medium-Range Weather Forecasts

[]{#_ENREF_64 .anchor}Matricardi, M., & Saunders, R. (1999). Fast radiative transfer model for simulation of infrared atmospheric sounding interferometer radiances. *Applied Optics, 38*, 5679-5691

[]{#_ENREF_65 .anchor}Matsunaga, T. (1994). A temperature-emissivity separation method using an empirical

relationship between the mean, the maximum, & the minimum of the thermal infrared

emissivity spectrum, in Japanese with English abstract. *Journal Remote Sensing Society Japan, 14*, 230-241

[]{#_ENREF_66 .anchor}Meerdink, S.K., Hook, S.J., Roberts, D.A., & Abbott, E.A. (2019). The ECOSTRESS spectral library version 1.0. *Remote Sensing of Environment, 230*

[]{#_ENREF_67 .anchor}Mesinger, F., DiMego, G., Kalnay, E., Mitchell, K., Shafran, P.C., Ebisuzaki, W., Jovic, D., Woollen, J., Rogers, E., Berbery, E.H., Ek, M.B., Fan, Y., Grumbine, R., Higgins, W., Li, H., Lin, Y., Manikin, G., Parrish, D., & Shi, W. (2006). North American regional reanalysis. *Bulletin of the American Meteorological Society, 87*, 343-+

[]{#_ENREF_68 .anchor}Mira, M., Valor, E., Boluda, R., Caselles, V., & Coll, C. (2007). Influence of soil water content on the thermal infrared emissivity of bare soils: Implication for land surface temperature determination. *Journal of Geophysical Research-Earth Surface, 112*, F04003

[]{#_ENREF_69 .anchor}Mushkin, A., & Gillespie, A.R. (2005). Estimating sub-pixel surface roughness using remotely sensed stereoscopic data. *Remote Sensing of Environment, 99*, 75-83

[]{#_ENREF_70 .anchor}Norman, J.M., & Becker, F. (1995). Terminology in Thermal Infrared Remote-Sensing of Natural Surfaces. *Agricultural and Forest Meteorology, 77*, 153-166

[]{#_ENREF_71 .anchor}Ogawa, K., Schmugge, T., & Rokugawa, S. (2006). Observations of the dependence of the thermal infrared emissivity on soil moisture. *Geophysical Research Abstracts, 8*, 04996

[]{#_ENREF_72 .anchor}Palluconi, F., Hoover, G., Alley, R.E., Nilsen, M.J., & Thompson, T. (1999). An atmospheric correction method for ASTER thermal radiometry over land, ASTER algorithm theoretical basis document (ATBD), Revision 3, Jet Propulsion Laboratory, Pasadena, CA, 1999

[]{#_ENREF_73 .anchor}Prata, A.J. (1994). Land-Surface Temperatures Derived from the Advanced Very High-Resolution Radiometer and the Along-Track Scanning Radiometer .2. Experimental Results and Validation of Avhrr Algorithms. *Journal of Geophysical Research-Atmospheres, 99*, 13025-13058

[]{#_ENREF_74 .anchor}Price, J.C. (1984). Land surface temperature measurements from the split window channels of the NOAA 7 Advanced Very High Resolution Radiometer. *Journal of Geophysical Research, 89*, 7231-7237

[]{#_ENREF_75 .anchor}Saunders, R., Matricardi, M., & Brunel, P. (1999). An improved fast radiative transfer model for assimilation of satellite radiance observations. *Quarterly Journal of the Royal Meteorological Society, 125*, 1407-1425

[]{#_ENREF_76 .anchor}Seemann, S.W., Borbas, E., Li, J., Menzel, P., & Gumley, L.E. (2006). MODIS Atmospheric Profile Retrieval Algorithm Theoretical Basis Document, Cooperative Institute for Meteorological Satellite Studies, University of Wisconsin-Madison, Madison, WI, Version 6, October 25, 2006

[]{#_ENREF_77 .anchor}Snyder, W.C., Wan, Z., Zhang, Y., & Feng, Y.Z. (1998). Classification-based emissivity for land surface temperature measurement from space. *International Journal of Remote Sensing, 19*, 2753-2774

[]{#_ENREF_78 .anchor}Strow, L.L., Hannon, S.E., Machado, S.D.S., Motteler, H.E., & Tobin, D.C. (2006). Validation of the Atmospheric Infrared Sounder radiative transfer algorithm. *Journal of Geophysical Research-Atmospheres, 111*

[]{#_ENREF_79 .anchor}Susskind, J., Barnet, C.D., & Blaisdell, J.M. (2003). Retrieval of atmospheric and surface parameters from AIRS/AMSU/HSB data in the presence of clouds. *Ieee Transactions on Geoscience and Remote Sensing, 41*, 390-409

[]{#_ENREF_80 .anchor}Teillet, P.M., Fedosejevs, G., Gautier, R.P., & Schowengerdt, R.A. (1998). Uniformity characterization of land test sites used for radiometric calibration of earth observation sensors. In, *Proc. 20th Can. Symp. Remote Sensing* (pp. 1-4). Calgary, AB, Canada

[]{#_ENREF_81 .anchor}Tonooka, H. (2001). An atmospheric correction algorithm for thermal infrared multispectral data over land - A water-vapor scaling method. *Ieee Transactions on Geoscience and Remote Sensing, 39*, 682-692

[]{#_ENREF_82 .anchor}Tonooka, H. (2005). Accurate atmospheric correction of ASTER thermal infrared imagery using the WVS method. *Ieee Transactions on Geoscience and Remote Sensing, 43*, 2778-2792

[]{#_ENREF_83 .anchor}Tonooka, H., Palluconi, F.D., Hook, S.J., & Matsunaga, T. (2005). Vicarious calibration of ASTER thermal infrared bands. *Ieee Transactions on Geoscience and Remote Sensing, 43*, 2733-2746

[]{#_ENREF_84 .anchor}Vaughan, R.G., Hook, S.J., Calvin, W.M., & Taranik, J.V. (2005). Surface mineral mapping at Steamboat Springs, Nevada, USA, with multi-wavelength thermal infrared images. *Remote Sensing of Environment, 99*, 140-158

[]{#_ENREF_85 .anchor}Wan, Z., & Li, Z.L. (2008). Radiance-based validation of the V5 MODIS land-surface temperature product. *International Journal of Remote Sensing, 29*, 5373-5395

[]{#_ENREF_86 .anchor}Wan, Z.M. (2008). New refinements and validation of the MODIS Land-Surface Temperature/Emissivity products. *Remote Sensing of Environment, 112*, 59-74

[]{#_ENREF_87 .anchor}Wan, Z.M., & Dozier, J. (1996). A generalized split-window algorithm for retrieving land-surface temperature from space. *Ieee Transactions on Geoscience and Remote Sensing, 34*, 892-905

[]{#_ENREF_88 .anchor}Wan, Z.M., & Li, Z.L. (1997). A physics-based algorithm for retrieving land-surface emissivity and temperature from EOS/MODIS data. *Ieee Transactions on Geoscience and Remote Sensing, 35*, 980-996

[]{#_ENREF_89 .anchor}Wang, K.C., & Liang, S.L. (2009). Evaluation of ASTER and MODIS land surface temperature and emissivity products using long-term surface longwave radiation observations at SURFRAD sites. *Remote Sensing of Environment, 113*, 1556-1565

[]{#_ENREF_90 .anchor}Watson, K. (1992). Spectral Ratio Method for Measuring Emissivity. *Remote Sensing of Environment, 42*, 113-116

[]{#_ENREF_91 .anchor}Watson, K., Kruse, F.A., & Hummermiller, S. (1990). Thermal Infrared Exploration in the Carlin Trend, Northern Nevada. *Geophysics, 55*, 70-79

[]{#_ENREF_92 .anchor}Yu, Y., Privette, J.L., & Pinheiro, A.C. (2008). Evaluation of split-window land surface temperature algorithms for generating climate data records. *Ieee Transactions on Geoscience and Remote Sensing, 46*, 179-192
