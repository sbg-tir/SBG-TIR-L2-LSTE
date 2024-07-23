# SBG-TIR OTTER Level 2 Surface Temperature & Emissivity (L2 LSTE) Data Product

[Glynn C. Hulley](https://github.com/ghulley) (he/him)<br>
[glynn.hulley@jpl.nasa.gov](mailto:glynn.hulley@jpl.nasa.gov)<br>
Lead developer and designer<br>
NASA Jet Propulsion Laboratory 329G

This repository will contain the Surface Biology and Geology Thermal Infrared (SBG-TIR) OTTER level 2 land-surface temperature and emissivity (L2 LSTE) data product algorithm.

JPL D-XXXXX

**Surface Biology and Geology (SBG) Observing Terrestrial Thermal Emission Radiometer (OTTER)**

##  1. Introduction

### 1.1. Identification

This is the Product Specification Document (PSD) for Level 2 (L2) data products of the Surface Biology and Geology (SBG) project. The SBG L2 products provide Land Surface Temperature and Emissivity (LST&E) and a Cloud Mask generated from data acquired by the SBG radiometer instrument according to the algorithm described in the SBG L2 LST&E Algorithm Theoretical Basis Document (ATBD) (JPL D-XXXXX) and L2 Cloud ATBD (JPL-D-XXXXX).

### 1.2. Purpose and Scope

This Product Specification Document (PSD) describes the standard Level 2 LSTE and Cloud Mask products generated in the SBG SDS at JPL. These include the detailed descriptions of the format and contents of the product and ancillary files that will be delivered to the Land Process Distributed Active Archive Center (LP-DAAC).

### 1.3.Mission Overview

### 1.4. Applicable and Reference Documents

"Applicable" documents levy requirements on the areas addressed in this document. "Reference" documents are identified in the text of this document only to provide additional information to readers. Unless stated otherwise, the document revision level is Initial Release. Document dates are not listed, as they are redundant with the revision level.

#### 1.4.1. Applicable Documents

#### 1.4.2. Reference Documents

### 1.5. SBG Data Products

The SBG mission will generate XX different distributable data products. The products represent four levels of data processing, with data granules defined as an image scene. Each image scene consists of XX scans of the instrument mirror, each scan taking approximately X seconds, and each image scene taking approximately XX seconds. Each image scene starts at the beginning of the first target area encountered during each orbit. Each orbit is defined \[insert orbit description\].

SBG Level 0 data include spacecraft packets that have been pre-processed by the Ground Data System (GDS). Level 1 products include spacecraft engineering data, the time-tagged raw sensor pixels appended with their radiometric calibration coefficients, the blackbody pixels used to generate the calibration coefficients, geolocated and radiometrically calibrated at-sensor radiances of each image pixel, the geolocation tags of each pixel, and the corrected spacecraft attitude data. Level 2 products include the land surface temperature and emissivity for each spectral band retrieved from the at-sensor radiance data, and a cloud mask. Level 2 products also appear in image scene granules. Level 3/4 products include plant functional traits, geology, and snow products derived from Level 2 products.

The SBG products are listed in Table 1-1. This document will discuss only the Level 2 products.

| **Product type** | **Description** | 
| L0A_FLEX | Level 0 "raw" spacecraft packets |
| L0A_HK |  Level 0 housekeeping packets |
| L1A_ENG | Spacecraft and instrument engineering data, including blackbody gradient coefficients |
| L1A_BB | Instrument Black Body calibration pixels |
| L1A_PIX | Raw pixel data with appended calibration coefficients |
| L1B_GEO | Geolocation tags, sun angles, and look angles, and calibrated, resampled at-sensor radiances |
| L1B_RAD | Radiometrically corrected, band-aligned, squared at-sensor radiance pixels |
| L2_LSTE | Land Surface temperature and emissivity |
| L2_CLOUD | Cloud mask |

*Table 1-1: SBG Distributable Standard Products*


##  Data Product OrganIzation

### Product File Format

The Network Common Data Form 4 (NetCDF-4) format will be used to distribute SBG granules at the orbit/scene level. These product files have a .nc file extension and are internally organized using the NetCDF-4 data standard. The NetCDF-4 rmat is utilized here for long-term archiving, and is not recommended for end-user analysis. These NetCDF-4 files are compatible with NetCDF Viewer, Panoply, and the NetCDF4 package in Python.

Information on Network Common Data Form (NetCDF-4) may be found at [[https://www.unidata.ucar.edu/software/netcdf/]{.underline}](https://www.unidata.ucar.edu/software/netcdf/).

#### File Level Metadata

All metadata that describe the full content of each granule of the SBG data product are stored within the explicitly named "/Metadata" Group. Metadata are handled using exactly the same procedures as those that are used to handle data. The contents of each Attribute that stores metadata conform to one of the SBG Types. Most metadata elements are stored as scalars. A few metadata elements are stored as arrays. The metadata appear in a set of HDF5 Groups under the "/Metadata" Group. These HDF5 Groups contain a set of HDF5 Attributes.

#### Local Metadata

SBG standards incorporate additional metadata that describe each HDF5 Dataset within the HDF5 file. Each of these metadata elements appear in an HDF5 Attribute that is directly associated with the HDF5 Dataset. Wherever possible, these HDF5 Attributes employ names that conform to the Climate and Forecast (CF) conventions. Table 2-3 lists the CF names for the HDF5 Attributes that SBG products typically employ.

| **CF Compliant Attribute Name** | **Description** | **Required?** | 
| --- | --- | --- |
| Units | Units of measure.  Appendix A lists applicable units for various data elements in this product. | Yes |
| valid_max | The largest valid value for any element in the Dataset.  The data type in valid_max matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding valid_max will also be float32. | No |
| valid_min | The smallest valid value for any element in the Dataset.  The data type in valid_min matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding valid_min will also be float32. | No |
| _FillValue | Specification of the value that will appear in the Dataset when an element is missing or undefined.  The data type of _FillValue matches the type of the associated Dataset.  Thus, if the associated Dataset stores float32 values, the corresponding _FillValue will also be float32. Datasets that do not have a fill value will omit this attribute. | No |
| long_name | A descriptive name that clearly describes the content of the associated Dataset. | Yes |

*Table 2-3: SBG Specific Local Attributes*

### Data Definition Standards 

The following sections of this document specify the characteristics and definitions of every data element stored in the SBG data products. Table 2-4 defines each of the specific characteristics that are listed in those sections. Some of these characteristics correspond with the SBG HDF5 Attributes that are associated with each Dataset. Data element characteristics that correspond to SBG HDF5 Attributes bear the same name. The remaining characteristics are descriptive data that help users better understand the data product content.

In some situations, a standard characteristic may not apply to a data element. In those cases, the field contains the character string 'n/a'. Hexadecimal representation sometimes indicates data content more clearly. Numbers represented in hexadecimal begin with the character string '0x'.

| **Characteristic** | **Definition** |
| --- | --- |
| Type | The data representation of the element within the storage medium. The storage class specification must conform to a valid SBG type. |
| Units | Units of measure.  Typical values include “deg”, “degC”, “Kelvin”, “meters/second”, “meters”, “m**2”, “seconds” and “counts”.  Appendix A includes references to important data measurement unit symbols. |

*Table 2-4: Data Element Characteristic Definitions*

#### Double Precision Time Variables

SBG double precision time variables contain measurements relative to the J2000 epoch. Thus, these variables represent a real number of Standard International (SI) compatible seconds since 11:58:55.816 on January 1, 2000 UTC.

#### Array Representation

This document employs array notation to demonstrate and clarify the correspondence among data elements in different product data elements. The array notation adopted in this document is similar to the standards of the Fortran programming language. Indices are one based. Thus, the first index in each dimension is one. This convention is unlike C or C++, where the initial index in each dimension is zero. In multidimensional arrays, the leftmost subscript index changes most rapidly. Thus, in this document, array elements ARRAY(15,1,5) and ARRAY(16,1,5) are stored contiguously.

HDF5 is designed to read data seamlessly regardless of the computer language used to write an application. Thus, elements that are contiguous using the dimension notation in this document will appear in contiguous locations in arrays for reading applications in any language with an HDF5 interface.

This document differentiates among array indices based on relative contiguity of storage of elements referenced with consecutive numbers in that index position. A faster or fastest moving index implies that the elements with consecutive numbers in that index position are stored in relative proximity in memory. A slower or slowest moving index implies that the elements referenced with consecutive indices are stored more remotely in memory. For instance, given array element ARRAY(15,1,5) in Fortran, the first index is the fastest moving index and the third index is the slowest moving index. On the other hand, given array element array\[4\]\[0\]\[14\] in C, the first index is the slowest moving index and the third index is the fastest moving index.

##  SBG Product Files

The SBG product file will contain at least 3 groups of data: A standard metadata group that specifies the same type of contents for all products, a product specific metadata group that specifies those metadata elements that are useful for defining attributes of the product data, and the group(s) containing the product data. (Note: A product metadata is not to be confused with a HDF5 object metadata.)

All product file names will have the form:

SBG_<PROD_TYPE>_<OOOOO>_<SSS>_<YYYYMMDD>T<hhmmss>_<BBBB>_<VV>.<TYPE>

Where:

PROD_TYPE:  Product type =
L2_LSTE, Land surface Temperature and Emissivity data
L2_CLOUD, Level 2 Cloud mask data

OOOOO:  Orbit number; starting at start of mission, ascending equatorial crossing
SSS:  Scene ID; starting at first scene of first orbit
YYYYMMDD:  Year, month, day of scene start time
hhmmss: Hour, minute, seconds of scene start time
BBBB:  Build ID of software that generated product, Major+Minor (2+2 digits)
VV:  Product version number (2 digits)
TYPE:  File type extension=
h5 for the data file
h5.met for the metadata file.

A SITE name is added to the ALEXI-USDA file name:
SBG_<PROD_TYPE>_<OOOOO>_<SSS>_<YYYYMMDD>T<hhmmss>_<BBbb>_<VV>.<SITE>.<TYPE>


### Standard Metadata

This is the minimal set of metadata that must be included with each product file. The standard metadata consists of the following:

| **Name** | **Type** | **Size** | **Example** |
| --- | --- | --- | --- |
| AncillaryInputPointer | String | variable | Group name of ancillary file list | 
| AutomaticQualityFlag | String | variable | PASS/FAIL (of product data) |
| BuildId | String | variable | |
| CollectionLabel | String | variable | |
| DataFormatType | String | variable | NCSAHDF5 |
| DayNightFlag | String | variable | |
| EastBoundingCoordinate | LongFloat | 8 | |
| HDFVersionId | String | variable | 1.8.16 |
| ImageLines | Int32 | 4 | 5632 |
| ImageLineSpacing | Float32 | 4 | 68.754 | 
| ImagePixels | Int32 | 4 | 5400 |
| ImagePixelSpacing | Float32 | 4 | 65.536 |
| InputPointer | String | variable | |
| InstrumentShortName | String | variable | SBG |
| LocalGranuleID | String | variable | --- |
| LongName | String | variable | SBG |
| InstrumentShortName | String | variable | --- |
| LocalGranuleID | String | variable | --- |
| LongName | String | variable | SBG |
| NorthBoundingCoordinate | LongFloat | 8 | --- |
| PGEName | String | variable | L2_LSTE (L2_CLOUD) |
| PGEVersion | String | variable | |
| PlatformLongName | String | variable | |
| PlatformShortName | String | variable | |
| PlatformType | String | variable | Spacecraft |
| ProcessingLevelID | String | variable | 1 |
| ProcessingLevelDescription | String | variable | Level 2 Land Surface Temperatures and Emissivity (Level 2 Cloud mask) |
| ProducerAgency | String | variable | JPL |
| ProducerInstitution | String | variable | Caltech |
| ProductionDateTime | String | variable | |
| ProductionLocation | String | variable | |
| CampaignShortName | String | variable | Primary |
| RangeBeginningDate | String | variable | |
| CampaignShortName | String | variable | |
| RangeBeginningDate | String | variable | |
| RangeBeginningTime | String | variable | |
| RangeEndingDate | String | variable | |
| RangeEndingTime | String | variable | |
| SceneID | String | variable | |
| ShortName | String | variable | L2_LSTE (L2_CLOUD) |
| SceneID | String | variable | |
| ShortName | String | variable | |
| SISName | String | variable | |
| SISVersion | String | variable | |
| SouthBoundingCoordinate | LongFloat | 8 | |
| StartOrbitNumber | String | variable | |
| StartOrbitNumber | String | variable | |
| WestBoundingCoordinate | LongFloat | 8 | |

*Table 3-1: Standard Product Metadata*


### Product-Specific Metadata

Any additional metadata necessary for describing the product will be recorded in this group.

#### L2 LSTE Metadata

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
*Table 3-2: L2 LSTE Metadata Definitions*

#### L2 CLOUD Metadata

| **Name** | **Type** | **Size** | **Example** | 
| --- | --- | --- | --- |
| QAPercentCloudCover | Int | 4 | 80 |
| CloudMeanTemperature | LongFloat | 8 | 231 |
| CloudMaxTemperature | LongFloat | 8 | 275 |
| CloudMinTemperature | LongFloat | 8 | 221 |
| CloudSDevTemperature | LongFloat | 8 | 0.45 |

*Table 3-3: L2 CLOUD Metadata Definitions*

###  Product Data

The product data will be stored in this group.

#### L2 LSTE data

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

#### L2 CLOUD data

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

### Low latency product

A low latency (\< 24 hour) product will be created in addition to the standard product. It may or may not be archived. The product contents will be the same as the standard product, although the method to obtain it will be different (see the relevant ATBD).

### Product Metadata File

The product metadata for each product file will be generated by the PCS from the metadata contents of each product file. The metadata will be converted into extensible markup language (XML). These will be used by the DAAC for cataloging. Exact contents and layout to be defined by PCS.

