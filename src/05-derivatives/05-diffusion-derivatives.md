# Diffusion derivatives

## Preprocessed diffusion weighted images

Multiple different versions of preprocessing can be stored for the same source
data. To distinguish them from each other, the `desc` fileneame keyword can be
used. Details of preprocessing performed for each variation of the processing
should be included in the pipeline documentation.

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
          <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_dwi.nii[.gz]
          <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_dwi.bvals
          <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_dwi.bvecs
```

The JSON sidecar file is REQUIRED (due to the REQUIRED `SkullStripped` field -
see [Common Data Types](02-common-data-types.md)), and if present can be used to
store information about what preprocessing options were used (for example
whether denoising was performed, corrections applied for field inhomogeneity /
gradient non-linearity / motion eddy currents, intensity normalization was
performed, etc.).

Additional reserved JSON metadata fields:

| **Key name**                   | **Description**                                                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Denoising                      | OPTIONAL. String. Denoising method                                                                                           |
| MotionCorrection               | OPTIONAL. Boolean. Motion correction                                                                                         |
| EddyCurrentCorrection          | OPTIONAL. Boolean. Eddy currents corrections                                                                                 |
| IntensityNormalizationMethod   | OPTIONAL. String. Method (if any) used for intensity normalization                                                           |
| FieldInhomogeneityCorrection   | OPTIONAL. Boolean. Correction for geometric distrotions arising from magnetic field inhomogeneity                            |
| GradientNonLinearityCorrection | OPTIONAL. String. Correction for non-linear gradients; allowed values: `none`, `geometry`, `gradients`, `geometry&gradients` |

## Diffusion Models (and input parameters)

Diffusion MRI can be modeled using various paradigms to extract a more easily
understandable representation of the diffusion process and the underlying
structure. To do so, parameters might be needed to control how the signal is fit
to the model. Those parameters are called **input parameters** in the following.
Once the model is fit, the resulting representation can be saved using a number
of values per voxel. Those values will be called **output** or **estimated
parameters** in the following.

**Estimated parameters** are saved as NIFTI files (see section Models for the
expected content of each model), and **input parameters** are saved in the
sidecar JSON file.

The following is a general example of naming convention:

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
            <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_model-<label>[_parameter-<parameter>]_diffmodel.nii[.gz]
            <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_model-<label>_diffmodel.json
```

The following models are codified and should be used in the `model-<label>`
field. If a new model is used, common sense should be used to derive a name
following the BIDS standard, and should ideally be integrated in a follow-up
version of the specification.

| Model label | Name & citation                                                                                | Description                                                                                                                                                                                                                                                                                                                                       |
| ----------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DTI`       | Diffusion tensor imaging (Basser et al. 1994)                                                  | 4D image with Dxx, Dxy, Dxz, Dyy, Dyz, Dzz; the 6 unique parameters of the diffusion tensor.                                                                                                                                                                                                                                                      |
| `DKI`       | Diffusion kurtosis imaging (Jensen et al., 2005)                                               | 4D image with Dxx, Dxy, Dxz, Dyy, Dyz, Dzz, Wxxxx, Wyyy, Wzzzz, Wxxxy, Wxxxz, Wxyyy, Wyyyz, Wxzzz, Wyzzz, Wxxyy, Wxxzz, Wyyzz, Wxxyz, Wxyyz, Wxyzz; Where D is the diffusion tensor and W is the kurtosis tensor.                                                                                                                                 |
| `WMTI`      | White matter tract integrity (Fieremans et al., 2011)                                          | 4D image with Dxx, Dxy, Dxz, Dyy, Dyz, Dzz, Wxxxx, Wyyy, Wzzzz, Wxxxy, Wxxxz, Wxyyy, Wyyyz, Wxzzz, Wyzzz, Wxxyy, Wxxzz, Wyyzz, Wxxyz, Wxyyz, Wxyzz, Dhxx, Dhxy, Dhxz, Dhyy, Dhyz, Dhzz, Drxx, Drxy, Drxz, Dryy, Dryz, Drzz, AWF; where D is the diffusion tensor, W is the kurtosis tensor, AWF is the additional axonal water fraction parameter |
| `CSD`       | Constrained Spherical Deconvolution (Tournier et al. 2007; Descoteaux et al. 2009)             | 4D image with spherical harmonic (SH) coefficients (number of volumes and their ordering depends on the model maximal degree and basis set, specified in the sidecar)                                                                                                                                                                                                                            |
| `NODDI`     | Neurite Orientation Dispersion and Density Imaging (Zhang et al. 2012, Daducci et al., 2015)   | 4D image with ICVF, OD, ISOVF; where ICVF is the “intracellular volume fraction” (also known as NDI), OD is the “orientation dispersion” (the variance of the Bingham; also known as ODI) and ISOVF is the isotropic component volume fraction (also known as IVF).                                                                               |
| `DSI`       | Diffusion Spectrum Imaging (Wedeen et al. 2008; Paquette et al 2017)                           | DSI is generally used mostly to compute the diffusion ODF (Orientation Distribution Function). No parameters are generally returned, but an ODF can be saved as a map.                                                                                                                                                                            |
| `CSA`       | Constant solid angle (Aganj et al. 2010)                                                       | 4D image with SH coefficients (number of volumes and their ordering depends on the model maximal degree and basis set, specified in the side-car)                                                                                                                                                                                                                           |
| `SHORE`     | Simple Harmonic Oscillator based Reconstruction and Estimation. (Ozarslan et al. 2008)         | 4D image with basis coefficients depending on choice of basis.                                                                                                                                                                                                                                                                                    |
| `MAPMRI`    | Mean Apparent Propagator MRI. (Ozarslan, 2013)                                                 | 4D image with basis coefficients depending on choice of basis.                                                                                                                                                                                                                                                                                    |
| `FORECAST`  | Fiber ORientation Estimated using Continuous Axially Symmetric Tensors. (Zuchelli et al. 2017) | 4D image with SH coefficients.                                                                                                                                                                                                                                                                                                                   |
| `fwDTI`     | Free water DTI (Hoy et al. 2014)                                                               | 4D image with D_c_xx, D_c_xy, D_c_xz, D_c_yy, D_c_yz, D_c_zz, FWF; where D_c is the fw-corrected diffusion tensor in this voxel.                                                                                                                                                                                                                  |
| `BedpostX`  | Ball-and-Stick model (Behrens et al. 2007; Jabdbi et al, MRM 2012)                             | 5D image (xyz, m, n) for m parameters and n MCMC samples (n=1 implies best-fit parameters). Parameters are f_i, th_i, ph_i (volume fraction, polar angle, azimuthal angle) for up to 3 fibres, d, d_std (for “multiexponential” model)                                                                                                            |

The JSON sidecar contains the following key/value pairs common for all models:

| **Key name**     | **Description**                                                                                                                      |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Shells           | OPTIONAL. Shells that were utilized to fit the model, as a list of b-values. If the key is not present, all shells were used.        |
| Gradients        | OPTIONAL. Subset of gradients utilized to fit the model, as a list of three-elements lists. If not present, all gradients were used. |
| ModelDescription | OPTIONAL. Extended information to describe the model.                                                                                |
| ModelURL         | OPTIONAL. URL to the implementation of the specific model utilized.                                                                  |
| Parameters       | OPTIONAL. A dictionary of model parameters (see below).                                                                              |

Parameters that may be stored within the JSON sidecar "Parameters" field depend on the particular model used:

-   `DTI` :

    -   `FitMethod` : {`WLS`,`OLS`,`NLLS`,`RESTORE`}
    -   `RESTORE` : sigma

-   `DKI` and `WMTI`:

    -   `FitMethod` : {`WLS`,`OLS`,`NLLS`}

-   `CSD`:

    -   `Tissue` : string
    -   `SphericalHarmonicDegree` : value
    -   `ResponseFunctionDegree` : value
    -   `ResponseFunction` : values
        (1 row per unique *b*-value, with lower *b*-values listed first;
        1 column per even harmonic degree starting from zero)
    -   `SphericalHarmonicBasis` : {`MRtrix 0.2`,`MRtrix3`,`DESCOTEAUX`}
    -   `NonNegativityConstraint` : {`soft`,`hard`,`none`}

-   `NODDI`:

    -   `DPar` : value
    -   `DIso` : value
    -   `Lambda1` : value
    -   `Lambda2` : value

-   `DSI` :

    -   `GridSize` : value
    -   `RStart` : value
    -   `RStep` : value
    -   `REnd` : value
    -   `FilterWidth` : value

-   `CSA` :

    -   `SphericalHarmonicOrder` : value
    -   `Smoothing` : value
    -   `Basis` : value

-   `SHORE` :

    -   `RadialOrder` : value
    -   `Zeta` : value
    -   `LambdaN` : value
    -   `LambdaL` : value
    -   `Tau` : value
    -   `ConstrainE0` : value
    -   `PositiveConstraint` : value
    -   `PosGrid` : value
    -   `PosRadius` : value

-   `MAPMRI` :

    -   `RadialOrder` : value
    -   `LaplacianRegularization` : bool
    -   `LaplacianWeighting` : value
    -   `PositivityConstraint` : bool
    -   `Tau` : value
    -   `ConstrainE0` : value
    -   `PositiveConstraint` : value
    -   `PosGrid` : value
    -   `PosRadius` : value
    -   `AnisotropicScaling` : bool
    -   `EigenvalueThreshold` : value
    -   `PosGrid` : value
    -   `BvalThreshold` : value
    -   `DTIScaleEstimation` : bool
    -   `StaticDiffusivity` : value

-   `FORECAST` :

    -   `Sphere` : value
    -   `DecAlg` : value
    -   `LambdaLb` : value
    -   `SphericalHarmonicsOrder` : value

-   `fwDTI` :

    -   `FitMethod` : {"WLS","NLLS"}

-   `BEDPOSTX` :

    -   `NFibers` : value
    -   `Fudge` : value
    -   `BurnIn` : value
    -   `NJumps` : value
    -   `SampleEvery` : value
    -   `Model` : {`monoexponential`,`multiexponentialStick`,
        `multiexponentialZeppelin`}

Examples:

A 4D volume with floating point numbers:

```Text
my_diffusion_pipeline/
    sub-01/
        dwi/
            sub-01_referenecemap-T1w_desc-WLS_model-DTI_diffmodel.nii.gz
```

```Text
my_diffusion_pipeline/
    sub-01/
        dwi/
            sub-01_referenecemap-T1w_desc-WLS_model-DTI_diffmodel.json
```

```JSON
{
    "Model": "DTI",
    "Parameters": {
        "FitMethod": "WLS"
    }
}
```

## Model-derived maps (output parameters)

Models output maps of estimated parameters. Commonly one parameter is estimated
per voxel; we call these maps of parameters. Two types of maps are described in
this specification: scalar and vector maps. Scalar maps are saved as 3D
.nii/nii.gz files. Vector maps are saved as 4D .nii/nii.gz files. Below the
specification for the naming of the files and a the list of currently accepted
fields for the maps.

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
            <source_keywords>[_referencemap-<referencemap>][_space-<space>][_desc-<label>]_model-<label>_<parameter>.nii[.gz]
```

### Scalar maps

These maps are saved 3D NIfTI files (x,y,z). Some examples of such maps are as follows:

| `<parameter>` value | Description                                                                                                                 | Possible Model sources                                                                                                       | Unit or scale                                                                                                                                                                        |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| FA                  | Fractional Anisotropy                                                                                                       | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   | Unitless \[0-1\]                                                                                                                                                                     |
| MD                  | Mean diffusivity (also called apparent diffusion coefficient, ADC)                                                          | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   | microns^2/ms (for example, for free water in body temperature, this should be approximately 3)                                                                                                     |
| AD                  | Axial Diffusivity (also called parallel diffusivity)                                                                        | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   | microns^2/ms (for example, for free water in body temperature, this should be approximately  3)                                                                                                     |
| RD                  | Radial Diffusivity (also called perpendicular diffusivity)                                                                  | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   | microns^2/ms (for example, for free water in body temperature, this should be approximately 3)                                                                                                     |
| MODE                | Mode of the tensor                                                                                                          | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   |                                                                                                                                                                                      |
| LINEARITY           | Tensor linearity (Westin 1997)                                                                                              | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   |                                                                                                                                                                                      |
| PLANARITY           | Tensor planarity (Westin 1997)                                                                                              | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   |                                                                                                                                                                                      |
| SPHERICITY          | Tensor sphericity (Westin 1997)                                                                                             | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   |                                                                                                                                                                                      |
| MK                  | Mean kurtosis                                                                                                               | DKI, WMTI                                                                                                                    | Unitless                                                                                                                                                                             |
| RK                  | Radial kurtosis                                                                                                             | DKI, WMTI                                                                                                                    | Unitless                                                                                                                                                                             |
| AK                  | Axial kurtosis                                                                                                              | DKI, WMTI                                                                                                                    | Unitless                                                                                                                                                                             |
| ICVF                | Intracellular volume fraction. This can be calculated from different models, so can have slightly different interpretation. | WMTI (AWF), NODDI                                                                                                            | Proportion \[0-1\]                                                                                                                                                                   |
| DECFA               | Directionally-encoded color (DEC) FA                                                                                        | DTI, DKI, WMTI (D, Dh, Dr)                                                                                                   | 3 8-bit floating point numbers that encode x,y,z of the principal diffusion direction in the voxel. The norm of the vector is the FA in the voxel.Need to set the NiFTI dtype to RGB |
| ODI                 | Orientation dispersion index                                                                                                | NODDI                                                                                                                        | Normalised to \[0,1\]                                                                                                                                                                |
| ISOVF               | Isotropic volume fraction                                                                                                   | NODDI                                                                                                                        | microns^2/ms (for example, for free water in body temperature, this should be 3)                                                                                                     |
| GFA                 | Generalized Fractional Anisotropy                                                                                           | CSA, CSD, SHORE, MAPMRI, Forecast (or any model that can be represented as an ODF)                                           | Proportion \[0 - 1\]                                                                                                                                                                 |
| INDEX               | Scalar maps from bedpostx                                                                                                   | FSUM (sum of partial volume fractions of the sticks)F_i (volume fraction of stick i)D (diffusivity)DSTD (std of diffusivity) | FSUM \[0 - 1\]F_i \[0 - 1\]D and DSTD microns^2/ms (for example, for free water in body temperature, this should be 3)                                                               |
| RTPP                | Return to the plane probability                                                                                             | MAPMRI                                                                                                                       | Probability \[0 - 1\]                                                                                                                                                                |
| RTAP                | Return to axis probability                                                                                                  | MAPMRI                                                                                                                       | Probability \[0 - 1\]                                                                                                                                                                |

### Vector-valued maps

These maps are saved 4D NIfTI files (x,y,z, 3\*n), where each triplet of volumes
encodes both a direction in 3D space and some quantitiative value (these are
sometimes referred to as "fixels"). The number of volumes in the image must be
3 times the maximal number of fixels that can appear in any voxel in the image
(since 3 volumes are required to represent the data for each individual fixel).
For voxels that contain less than this number of fixels, the excess volumes
should contain either zero-filling, or NaN values. Exactly what is encoded by
the amplitude of each XYZ triplet depends on the underlying model properties
that were used to derive the map, with some examples being:

| `<map_label>` values | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| PDF                  | Diffusion propagator                                        |
| EVECS                | Eigenvectors of a model that has eigenvectors (such as DTI) |
| PEAKS                | Directions and amplitudes of ODF maxima on the sphere       |

## Describing tractography

Tractography normally generates one of two primary file types: tractograms or
NIfTi files containing maps. Tractograms are files containing a collection of
streamlines, which are objects describing the path of idealized brain fiber
fascicles (ensembles of neural fibers that travel together). Tractography
visitation maps contain 3D spatial histograms of the number / fraction of
streamlines intersecting each voxel, but do not contain the individual
streamlines trajectories themselves. Two tractogram file formats are supported;
tractography visitation maps should be saved as NIfTIs.

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
            <source_keywords>[_referencemap-<referencemap>][_desc-<label>][_subset-<label>]_tractography.[trk|tck|nii[.gz]]
```

### Filename keys

`desc` (optional) – A way to refer to a specific instance of the tractography
process. The combination of tractography methods and parameters that were used
to create this tractography result should be described in the associated JSON
file.

`subset` (optional) – A label descriptor for the the subset of streamlines
included in this file; if not specified, a whole brain tractography is assumed
(a tractogram). Example of subsets can be `short` if only short streamlines were
kept, or `subsampled50` if only 50% of the streamlines were kept.

### File formats

-   [.trk - Diffusion Toolkit (+TrackVis)](https://web.archive.org/web/20190103230122/http://www.trackvis.org/docs/?subsect=fileformat)
-   [.tck - MRtrix](https://web.archive.org/web/20190103230209/https://mrtrix.readthedocs.io/en/latest/getting_started/image_data.html)

### JSON Metadata

We have two main classes of tractography algorithms: Global or Local
tractography, each with various supported algorithms.

| Class  | Algorithms | Reference                                                                                                                            |
| ------ | ---------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| LOCAL  | PROB       | Sherbondy, A.J., et al. (2008); Tournier J.D. et al. (2012); Behrens et al. (2007)                                                   |
| LOCAL  | DET        | Conturo, T. et al. (1999); Mori, S. et al. (1999)                                                                                    |
| LOCAL  | EUDX       | Garyfallidis et al. (2010); Mori, S. et al. (1999)                                                                                   |
| LOCAL  | FACT       | Jiang, H. et al. (2005)                                                                                                              |
| LOCAL  | STT        | Lazar, M. et al. (2003)                                                                                                              |
| LOCAL  | NULL       | Morris et al. (2008)                                                                                                                 |
| GLOBAL | UKF        | Malcolm J-G. et al. (2009)                                                                                                           |
| GLOBAL | SpinGlass  | Fillard P. et al. (2009)                                                                                                             |
| GLOBAL | ENS        | Takemura et al. (2016)                                                                                                               |
| GLOBAL | Other      | Mangin, J.F. et al. (2013); Aganj, I. et al. (2011); Neher, P.F. et al. (2012); Jbabdi, S., et al. (2007); Reisert, M. et al. (2011) |

The JSON sidecar contains the following key/value pairs:

| **Key name**       | **Description**                                                                                                      |
| ------------------ | -------------------------------------------------------------------------------------------------------------------- |
| TractographyClass  | REQUIRED. Allowed values: `local`, `global`                                                                          |
| TractographyMethod | REQUIRED. Allowed values: `probabilistic`,`deterministic`,`eudx`,`fact`,`stt`,`null`,`ukf`,`spinglass`,`ens`,`other` |
| Description        | OPTIONAL. string                                                                                                     |
| Parameters         | OPTIONAL. A dictionary of model parameters (see below).                                                              |

RECOMMENDED to appear within the "Parameters" field when TractographyMethod is `probabilistic`, `deterministic`, `eudx`,
`fact`, `stt`, `null`:

| **Key name**             | **Description**                                               |
| ------------------------ | ------------------------------------------------------------- |
| StepSizeUnits            | "mm","norm"                                                   |
| StepSize                 | value                                                         |
| AngleCurvature           | value                                                         |
| RadiusCurvature          | value                                                         |
| MinimumLength            | value                                                         |
| MaximumLength            | value                                                         |
| IntegrationOrder         | integer                                                       |
| Unidirectional           | bool                                                          |
| SeedingMethod            | "seedsPerVoxel", "globalSeedNumber", "totalStreamlinesNumber" |
| SeedingNumber            | integer                                                       |
| TerminationCriterion     | "leaveMask", "reachingTissueType”                             |
| TerminationCriterionTest | "binary", "ACT", "CMC"                                        |

Example:

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
             sub-01_desc-DET1_tractography.trk
             sub-01_desc-DET1_tractography.json
```

```JSON
{
    "TractographyClass": "local",
    "TractographyMethod": "deterministic",
    "Parameters": {
        "StepSize": 0.5
    }
}
```

```Text
<pipeline_name>/
    sub-<participant_label>/
        dwi/
            sub-01_subset-shortfibers_tractography.trk
            sub-01_subset-shortfibers_tractography.json
```

```JSON
{
    "TractographyClass": "global",
    "TractographyMethod": "UKF",
    "Description": "UKF tracking where only short fibers were kept."
    "Parameters": {
        "MaximumLength": 50.0
    }
}
```
