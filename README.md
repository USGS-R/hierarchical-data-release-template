# hierarchical-data-release-template

## code

Need to have non-CRAN packages`dssecrets`, `meddle`, and `scipiper` installed, among other packages. 
Need to have CRAN package `sbtools` installed

- Create a data release or create a child item on sciencebase to experiment on
- Add "write" permissions on the release item on sciencebase for `cidamanager` (this is the `dssecrets` service account)
- Change the files and the functions in `src/` to what you need
- Edit data release information in `in_text/text_data_release.yml` to fit your data release and your file names and contents
- modify the sciencebase indentifier to your parent data release identifier (should be a string that is something like "5faaac68d34eb413d5df1f22")
- run `scmake()`
- validate your `out_xml/fgdc_metadata.xml` file with the [validator tool](https://mrdata.usgs.gov/validation/)
- fix any validation errors (usually this requires filling in metadata information in the `in_text/text_data_release.yml` and perhaps looking a the [metadata template](https://raw.githubusercontent.com/USGS-R/meddle/master/inst/extdata/FGDC_template.mustache))
- win

## Repo File Details

### Data processing scripts

* 1_spatial.yml: outlines processing steps and output targets of spatial data

* 2_observations.yml: outlines processing steps and output targets for observations data

* 3_drivers.yml: outlines processing steps and output targets for model driver data

* 4_forecasts.yml: outlines processing steps and output targets for forecasts

### Metadata 

All Science Base items and child items should have an associated metadata page build in a yaml. They are saved in `in_text` folder and should match the respective parent and/or child data yamls. E.g. the metadata file for the output files stored in 1_spatial.yml is in `./in_text/text_01_spatial.yml`.

Please build your metadata work off the pre-populated metadata templates that meet the fgdc structure and map to the expected xml on science base. 

### Remake

* Remake.yml
The purpose of the remake yaml file assembles are relevant targets and uploads  to Science Base (much like a `_targets.R` file in a targets pipeline).
Unlike the flat structure,  data munging, manipulation, and file writing happens in the child item yml in the hierarchical data release repo structure. However, additional processing can also happen in the remake file in addition to the ScienceBase uploads.


* Two ways to push to Science Base: 

Remake yml is the single source for uploads to sciencebase, and there are two ways to do this in the template. 
The more simple method pushes the data and metadata in two separate targets using and `XX_sb_data` `XX_sb_xml` via functions `sb_render_post_xml()` (transforms to xml and pushes to SB) and `sb_replace_files` (pushes datafiles to SB)

The second method uploads data and the xml (metadata) and data at the same time. This upload approach uses an internal task table (log) where you can specify all files that should be pushed to the same sbid at one time. Again, because the upload step uses an internal task table, data files aren't replaced everytime you fix a metadata typo or add information to a metadata field. The result of the sciencebase push step is a file with timestamps for when each file got pushed that can be checked into GitHub. Having the file with timestamps in GitHub will clearly show when updates were made and will render nicely without having to build the object target locally.

Depending on what approach you choose, make sure to comment out the irrelevant code chunks and targets:

```yaml
targets:
  all:
    depends:
      - 00_parent_sb_xml
      
      - 01_spatial_sb_xml
      - 01_spatial_sb_data
#      - log/01_spatial_sb_data.csv
      
```
For simple data uploads to SB without building a log file, use `sb_replace_files()` function from `src/sb_utils.R`. Two parameters are required by this function: the `sb_id`, and the data file(s) to publish, that are stored in the local `out_data` folder.

To push the files to ScienceBase with the log table, use the `sb_replace_files_log()` function from `src/sb_utils.R` of this repo template. If you are uploading many files at once using `sb_replace_files_log` (either in a file hash or just multiple files passed in through `...`), it is recommended to use a task table to do so. Internal task table methods are enabled by default. If you do not want to use an internal task table, set the `use_task_table = FALSE` when using `sb_replace_files`. You must also specify the file(s) where the `sb_replace_files_log` functions exist using the argument, `sources`. You can specify multiple files and a file hash file in one call to `sb_replace_files_log`. Currently, each `sb_replace_files` functions can only push to one sbid. 

E.g.
```yaml
   log/01_spatial_sb_data.csv:
     command: sb_replace_files_log(
       filename = target_name,
       sb_id = sbid_01_spatial,
       sources = "src/sb_utils.R",
       "out_data/XX_geospatial_area_WG84.zip",
       "out_xml/01_spatial.xml")
```


