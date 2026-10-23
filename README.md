# Met Éireann Climate Data Downloader for R

An R function for downloading, cleaning, and filtering historical
climate data directly from the **Met Éireann Climate Database**.

The `download_met_data()` function provides a convenient workflow for
retrieving **hourly, daily, or monthly weather observations** from Met
Éireann climate stations in Ireland.

## Features

`download_met_data()` can:

-   Download hourly, daily, or monthly Met Éireann climate data
-   Select stations using their Met Éireann station ID
-   Validate whether the requested station and data frequency are
    available
-   Download and cache the station database
-   Automatically identify the data header in Met Éireann CSV files
-   Clean and parse dates and timestamps
-   Convert measurement variables to numeric format
-   Filter observations to a specified date range
-   Handle monthly date filtering at the calendar-month level
-   Generate sequential day numbers for hourly and daily datasets
-   Attach station and download information as metadata
-   Retry failed downloads automatically
-   Provide informative progress and error messages

## Requirements

The function uses:

``` r
library(dplyr)
library(lubridate)
library(httr2)
```

Install them if required:

``` r
install.packages(c("dplyr", "lubridate", "httr2"))
```

The `writexl` package is optional and can be used to export results to
Excel:

``` r
install.packages("writexl")
library(writexl)
```

## Function

The main function is:

``` r
download_met_data()
```

### Arguments

  -----------------------------------------------------------------------
  Argument                            Description
  ----------------------------------- -----------------------------------
  `timeline`                          Data frequency: `"hourly"`,
                                      `"daily"`, or `"monthly"`.

  `start_date`                        Start date as a `Date` object or
                                      `"YYYY-MM-DD"`.

  `end_date`                          End date as a `Date` object or
                                      `"YYYY-MM-DD"`.

  `station`                           Met Éireann station ID, e.g. `1775`
                                      or `"1775"`.

  `timeout_sec`                       Maximum time allowed for each
                                      download request. Default = `60`
                                      seconds.

  `max_retries`                       Maximum number of download
                                      attempts. Default = `3`.

  `refresh_cache`                     If `TRUE`, downloads a fresh copy
                                      of the station database. Default =
                                      `FALSE`.

  `verbose`                           If `TRUE`, displays progress and
                                      completion messages. Default =
                                      `TRUE`.
  -----------------------------------------------------------------------

## Basic Usage

### Hourly Data

``` r
Hourly_data <- download_met_data(
  station = 1775,
  timeline = "hourly",
  start_date = "2024-11-15",
  end_date = "2025-04-05"
)
```

Hourly output begins with:

``` text
Timestamp | Date | Hour | Days | ...
```

### Daily Data

``` r
Daily_data <- download_met_data(
  station = 1775,
  timeline = "daily",
  start_date = "2024-11-15",
  end_date = "2025-04-05"
)
```

Daily output includes:

``` text
Date | Days | ...
```

### Monthly Data

``` r
Monthly_data <- download_met_data(
  station = 1775,
  timeline = "monthly",
  start_date = "2024-11-15",
  end_date = "2025-04-05"
)
```

Monthly output begins with:

``` text
Date | year | month | Month Name | ...
```

Monthly filtering is performed at the **calendar-month level**. The
example above therefore includes November 2024 through April 2025 even
though the supplied dates occur partway through those months.

## Partial Date Ranges

Both dates are optional.

``` r
Daily_data <- download_met_data(
  station = 1775,
  timeline = "daily",
  start_date = "2025-01-01"
)
```

If `end_date` is omitted, the function uses the current date.

``` r
Daily_data <- download_met_data(
  station = 1775,
  timeline = "daily",
  end_date = "2025-01-01"
)
```

If only `end_date` is supplied, filtering begins at `1900-01-01`. If
neither date is supplied, all available observations returned by Met
Éireann are retained.

## Station Database

The function validates the requested station against a station database
containing station number, station name, county, and data-frequency
availability.

The station database used by the function is available in this
repository as:

`Met Stations.csv`

It contains fields such as:

``` text
Station Number
Station Name
County
Minutely
Hourly
Daily
Monthly
```

The database is downloaded once and cached during the R session. To
force a fresh download:

``` r
Daily_data <- download_met_data(
  station = 1775,
  timeline = "daily",
  refresh_cache = TRUE
)
```

The current function supports **hourly, daily, and monthly** climate
data.

## Station and Timeline Validation

Before downloading climate observations, the function checks whether the
station ID exists and whether the requested timeline is available for
that station.

``` r
download_met_data(
  station = 1775,
  timeline = "hourly"
)
```

If the requested frequency is unavailable, the function stops and
reports the frequencies available for that station.

## Hourly Weather Variables

  Variable   Description                  Unit
  ---------- ---------------------------- ---------
  `date`     Date and time                UTC
  `rain`     Precipitation amount         mm
  `temp`     Air temperature              °C
  `wetb`     Wet bulb temperature         °C
  `dewpt`    Dew point temperature        °C
  `vappr`    Vapour pressure              hPa
  `rhum`     Relative humidity            \%
  `msl`      Mean sea level pressure      hPa
  `wdsp`     Mean wind speed              kt
  `wddir`    Predominant wind direction   degrees

The original date/time field is converted into `Timestamp`, `Date`, and
`Hour`. A `Days` variable is generated from the sequential rank of each
unique date.

## Daily Weather Variables

  Variable   Description                                      Unit
  ---------- ------------------------------------------------ ---------
  `date`     Observation date                                 UTC
  `rain`     Precipitation amount                             mm
  `maxtp`    Maximum air temperature                          °C
  `mintp`    Minimum air temperature                          °C
  `gmin`     Grass minimum temperature                        °C
  `soil`     Mean 10 cm soil temperature                      °C
  `cbl`      Mean CBL pressure                                hPa
  `wdsp`     Mean wind speed                                  kt
  `hm`       Highest 10-minute mean wind speed                kt
  `ddhm`     Wind direction at maximum 10-minute mean         degrees
  `hg`       Highest gust                                     kt
  `pe`       Potential evapotranspiration                     mm
  `evap`     Evaporation                                      mm
  `smd_wd`   Soil moisture deficit, well-drained soil         mm
  `smd_md`   Soil moisture deficit, moderately drained soil   mm
  `smd_pd`   Soil moisture deficit, poorly drained soil       mm
  `glorad`   Global radiation                                 J cm⁻²

## Monthly Weather Variables

  Variable   Description                 Unit
  ---------- --------------------------- -------
  `year`     Year                        
  `month`    Month                       
  `rain`     Precipitation amount        mm
  `meant`    Mean air temperature        °C
  `maxtp`    Maximum air temperature     °C
  `mintp`    Minimum air temperature     °C
  `mnmax`    Mean maximum temperature    °C
  `mnmin`    Mean minimum temperature    °C
  `gmin`     Grass minimum temperature   °C
  `wdsp`     Mean wind speed             kt
  `mxgt`     Highest gust                kt
  `sun`      Sunshine duration           hours

The function also creates `Date` and `Month Name`. The monthly `Date`
represents the first day of the corresponding calendar month.

## Metadata

Download information is attached to the returned data frame as a
`metadata` attribute:

``` r
attr(Hourly_data, "metadata")
```

Metadata include:

``` text
station_id
station_name
county
timeline
period_start
period_end
downloaded
source
download_url
function_version
```

Example:

``` r
metadata <- attr(Hourly_data, "metadata")

metadata$station_name
metadata$county
metadata$period_start
metadata$period_end
```

## Exporting Data to Excel

A single dataset can be exported using `writexl`:

``` r
writexl::write_xlsx(
  Hourly_data,
  "Hourly_Weather_Data.xlsx"
)
```

Multiple datasets can be saved in separate worksheets:

``` r
Met_data <- list(
  "Hourly Data" = Hourly_data,
  "Daily Data" = Daily_data,
  "Monthly Data" = Monthly_data
)

writexl::write_xlsx(
  Met_data,
  "Met_Eireann_Weather_Data.xlsx"
)
```

## Example Workflow

``` r
Hourly_data <- download_met_data(
  station = 1775,
  timeline = "hourly",
  start_date = "2024-11-15",
  end_date = "2025-04-05"
)

head(Hourly_data)
summary(Hourly_data)
attr(Hourly_data, "metadata")
```

## Error Handling

The function checks for common problems including:

-   Invalid station IDs
-   Stations not present in the station database
-   Unavailable hourly, daily, or monthly data
-   Invalid date ranges
-   Failed internet requests
-   Empty downloads
-   Unexpected CSV structures
-   Missing required variables
-   Dates or timestamps that cannot be parsed

Download requests also use timeout and retry handling to improve
reliability.

## Data Source

Climate observations are obtained from the **Met Éireann Climate
Database**.

Met Éireann remains the original provider and owner of the climate data.
Users should consult Met Éireann for the applicable data terms,
documentation, and guidance when using these observations in research or
other applications.

## Version

Current function version:

``` text
download_met_data() Version 4.4
```

## Author

**Benjamin Macartan**

## Citation

If this code is used in research, reports, or publications, please
acknowledge the original climate data source as **Met Éireann**.

A formal citation for this R code can be added if the repository is
archived and assigned a DOI in the future.

## License

No software license has yet been specified for this repository.

If the code is intended for public reuse, an open-source license such as
the **MIT License** can be added to the repository.
