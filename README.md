# Tableau Scraper

[![PyPI](https://img.shields.io/pypi/v/TableauScraper.svg)](https://pypi.python.org/pypi/TableauScraper)
[![CI](https://github.com/bertrandmartel/tableau-scraping/workflows/CI/badge.svg)](https://github.com/bertrandmartel/tableau-scraping/actions)
[![codecov](https://codecov.io/gh/bertrandmartel/tableau-scraping/branch/master/graph/badge.svg?token=F4R3NZF796)](https://codecov.io/gh/bertrandmartel/tableau-scraping)
[![License](http://img.shields.io/:license-mit-blue.svg)](LICENSE.md)

Python library to scrape data from [Tableau viz](https://public.tableau.com/fr-fr/gallery)

R library is under development but a script is available to get the worksheets, see [this](https://github.com/bertrandmartel/tableau-scraping#r)

## Python

### Install

```bash
pip install TableauScraper
```

### Usage

- Get worksheets data

```python
from tableauscraper import TableauScraper as TS

url = "https://public.tableau.com/views/PlayerStats-Top5Leagues20192020/OnePlayerSummary"

ts = TS()
ts.loads(url)
workbook = ts.getWorkbook()

for t in workbook.worksheets:
    print(f"worksheet name : {t.name}") #show worksheet name
    print(t.data) #show dataframe for this worksheet
```

[Try this on repl.it](https://repl.it/@bertrandmartel/TableauGetWorksheets)

- Get a specific worksheet

```python
from tableauscraper import TableauScraper as TS

url = "https://public.tableau.com/views/PlayerStats-Top5Leagues20192020/OnePlayerSummary"

ts = TS()
ts.loads(url)

ws = ts.getWorksheet("ATT MID CREATIVE COMP")
print(ws.data)
```

- select a selectable item

```python
from tableauscraper import TableauScraper as TS

url = "https://public.tableau.com/views/PlayerStats-Top5Leagues20192020/OnePlayerSummary"

ts = TS()
ts.loads(url)

ws = ts.getWorksheet("ATT MID CREATIVE COMP")

# show selectable values
selections = ws.getSelectableItems()
print(selections)

# select that value
dashboard = ws.select("ATTR(Player)", "Vinicius Júnior")

# display worksheets
for t in dashboard.worksheets:
    print(t.data)
```

[Try this on repl.it](https://repl.it/@bertrandmartel/TableauSelectItem)

- set parameter

Get list of parameters with `workbook.getParameters()` and set parameter value using `workbook.setParameter("column_name", "value")` :

```python
from tableauscraper import TableauScraper as TS

url = "https://public.tableau.com/views/PlayerStats-Top5Leagues20192020/OnePlayerSummary"

ts = TS()
ts.loads(url)
workbook = ts.getWorkbook()

# show parameters values / column
parameters = workbook.getParameters()
print(parameters)

# set parameters column / value
workbook = workbook.setParameter("P.League 2", "Ligue 1")

# display worksheets
for t in workbook.worksheets:
    print(t.data)
```

[Try this on repl.it](https://repl.it/@bertrandmartel/TableauParameter)

- set filter

Get list of filters with `worksheet.getFilters` and set filter value using `worksheet.setFilter("column_name", "value")`:

```python
from tableauscraper import TableauScraper as TS

url = 'https://public.tableau.com/views/WomenInOlympics/Dashboard1'
ts = TS()
ts.loads(url)

# show original data for worksheet
ws = ts.getWorksheet("Bar Chart")
print(ws.data)

# get filters columns and values
filters = ws.getFilters()
print(filters)

# set filter value
wb = ws.setFilter('Olympics', 'Winter')

# show the new data for worksheet
countyWs = wb.getWorksheet("Bar Chart")
print(countyWs.data)
```

[Try this on repl.it](https://repl.it/@bertrandmartel/TableauFilter)

- Go to sheet

Get list of all sheets with subsheets visible or invisible, ability to send a go-to-sheet command (dashboar button) :

```python
from tableauscraper import TableauScraper as TS

url = "https://public.tableau.com/views/COVID-19VaccineTrackerDashboard_16153822244270/Dosesadministered"
ts = TS()
ts.loads(url)
workbook = ts.getWorkbook()

sheets = workbook.getSheets()
print(sheets)

nycAdults = workbook.goToSheet("NYC Adults")
for t in nycAdults.worksheets:
    print(f"worksheet name : {t.name}")  # show worksheet name
    print(t.data)  # show dataframe for this worksheet
```

### Sample usecases

- https://replit.com/@bertrandmartel/TableauOregonCovid
- https://replit.com/@bertrandmartel/TableauCovidIndia
- https://replit.com/@bertrandmartel/TableauCovidArizona
- https://replit.com/@bertrandmartel/TableauIllinoisOpioId
- https://replit.com/@bertrandmartel/TableauCovidNY
- https://replit.com/@bertrandmartel/TableauCovidNCDHHS
- https://replit.com/@bertrandmartel/TableauCovidWisconsin
- https://replit.com/@bertrandmartel/TableauScrapeNewspaper
- https://replit.com/@bertrandmartel/TableauStoryPoints
- https://replit.com/@bertrandmartel/TableauCovidOhio
- https://replit.com/@bertrandmartel/TableauCovidSouthCarolina
- https://replit.com/@bertrandmartel/TableauCovidNewHampshire
- https://replit.com/@bertrandmartel/TableauCovidNewJersey
- https://replit.com/@bertrandmartel/TableauCovid19Wyoming
- https://replit.com/@bertrandmartel/TableauCovid19Louisiana
- https://replit.com/@bertrandmartel/TableauCIESFootball
- https://replit.com/@bertrandmartel/TableauCovid19TestingCommonsASU

### Server side rendering

If the tableau url you're working on is using [server side rendering](https://help.tableau.com/current/server/en-us/browser_rendering.htm), data can't be extracted as is.

You can checkout if your tableau url is using server side rendering by opening chrome development console / network tab. You would notice those API calls when mouse hovering tables or maps `render-tooltip-server`:

![tooltip](https://user-images.githubusercontent.com/5183022/115800554-2642a800-a3db-11eb-9a71-e279cbc58aaf.png)

Server side rendering means that no data is sent to the browser. Instead, the server is rendering the tableau chart using images only and detects selection using mouse coordinates.

To extract the data, one thing that has worked with some tableau url was to trigger a specific filter that is not server-side-rendered. You can checkout the network tab on Chrome development console to check if the filter call is using or not server-side rendering or client-side-rendering with `renderMode`:

![client side rendering](https://user-images.githubusercontent.com/5183022/115800868-b7198380-a3db-11eb-95c0-7104f7f0c77c.png)

If the filter is only using client side rendering, you can list all filters and perform the filter for each value. This technique only works if the tableau data has "cleared" the filter by default otherwise the data is already cached when the tableau data is loaded, and since it's using server side rendering you can't access this data

Checkout the following repl.it for examples with tableau url using server side rendering:

- https://replit.com/@bertrandmartel/TableauCIESFootball
- https://replit.com/@bertrandmartel/TableauCovid19TestingCommonsASU

### Testing Python script

To discover all worksheets, selectable columns and dropdowns, run `prompt.py` script under `scripts` directory :

```bash
git clone git@github.com:bertrandmartel/tableau-scraping.git
cd tableau-scraping/scripts

#get worksheets data
python3 prompt.py -get workbook -url "https://public.tableau.com/views/COVID-19inMissouri/COVID-19inMissouri"

#select a selectable item
python3 prompt.py -get select -url "https://public.tableau.com/views/MKTScoredeisolamentosocial/VisoGeral"

#set a parameter
python3 prompt.py -get parameter -url "https://public.tableau.com/views/COVID-19DailyDashboard_15960160643010/Casesbyneighbourhood"
```

### Settings

`TableauScraper` class has the following optional parameters :

| Parameters | default value | description                                                       |
| ---------- | ------------- | ----------------------------------------------------------------- |
| logLevel   | logging.INFO  | log level                                                         |
| delayMs    | 500           | minimum delay in millis between actions (select/dropdown request) |

## R

under `R` directory :

```R
Rscript tableau.R
```

R library is under development

## Stackoverflow Questions

See [those stackoverflow posts about this topic](https://stackoverflow.com/search?q=user%3A2614364+%5Btableau-api%5D)
