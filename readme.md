![PyPI - Downloads](https://img.shields.io/pypi/dm/datamule)
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fjohn-friedman%2Fdatamule-python&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
![GitHub](https://img.shields.io/github/stars/john-friedman/datamule-python)

# datamule
A python package to make using SEC filings easier. Integrated with [datamule](https://datamule.xyz/)'s APIs and datasets. Other useful SEC packages include:
[Edgartools](https://github.com/dgunning/edgartools), [SEC Parsers](https://github.com/john-friedman/SEC-Parsers).

<b>NOTICE</b>: As of v0.29 Downloader no longer requires indices.

## features
* monitor edgar for new filings
* parse textual filings into simplified html, interactive html, or structured json.
* download sec filings quickly and easily
* download datasets such as every MD&A from 2024 or every 2024 10K converted to structured json

## installation

installation
```
pip install datamule
```

installation with filing_viewer module
```
pip install datamule['filing_viewer']
```

## quickstart:
```
import datamule as dm
downloader = dm.Downloader()
downloader.download(form='10-K',ticker='AAPL')
```

## documentation

### downloader
Uses the [EFTS API](https://efts.sec.gov/LATEST/search-index?&ciks=0001080306&forms=10-K&startdt=2001-01-01&enddt=2024-09-17). [Documentation here](https://github.com/john-friedman/datamule-python/blob/main/endpoints/efts.md).

```
downloader = dm.Downloader()
```

### downloads 

`downloader.download()` downloads filings.

```
# Example 1: Download all 10-K filings for Tesla using CIK
downloader.download(form='10-K', cik='1318605', output_dir='filings')

# Example 2: Download 10-K filings for Tesla and META using CIK
downloader.download(form='10-K', cik=['1318605','1326801'], output_dir='filings')

# Example 3: Download 10-K filings for Tesla using ticker
downloader.download(form='10-K', ticker='TSLA', output_dir='filings')

# Example 4: Download 10-K filings for Tesla and META using ticker
downloader.download(form='10-K', ticker=['TSLA','META'], output_dir='filings')

# Example 5: Download every form 3 for a specific date
downloader.download(form ='3', date='2024-05-21', output_dir='filings')

# Example 6: Download every 10K for a year
downloader.download(form='10-K', date=('2024-01-01', '2024-12-31'), output_dir='filings')

# Example 7: Download every form 4 for a list of dates
downloader.download(form = '4',date=['2024-01-01', '2024-12-31'], output_dir='filings')
```

If `return_urls` is set to True, returns filing primary document urls instead of downloading them. Note that `human_readable` option has been removed. If you want this back, let me know.

`downloader.watch()` checks the SEC for new filings. Can set frequency as low as 100 milliseconds.

watching all companies and forms
```
print("Monitoring SEC EDGAR for changes...")
changed_bool = indexer.watch(1,silent=False)
if changed_bool:
    print("New filing detected!")
```

watching specific companies and forms
```
print("Monitoring SEC EDGAR for changes...")
changed_bool = indexer.watch(1,silent=False,cik=['0001267602','0001318605'],form=['3','S-8 POS'])
if changed_bool:
    print("New filing detected!")
```

### download_datasets
```
downloader.download_dataset('10K') # Every 2024 10-K up to September 1st converted to json form. 
downloader.download_dataset('MDA') # Every MD&A extracted from the 10K dataset.
```

Need a better way to store datasets, as I'm running out of storage. Currently stored on [Dropbox](https://www.dropbox.com/scl/fo/byxiish8jmdtj4zitxfjn/AAaiwwuyaYp_zRfFyqfBUS8?rlkey=sx7g5uxrz4dn35c593584ztds&st=yohhlwfx&dl=0) 2gb free tier.


## parsing
Uses endpoint: https://jgfriedman99.pythonanywhere.com/parse_url with params url and return_type. Current endpoint can be slow. If it's too slow for your use-case, please contact me.

### simplified html
```
simplified_html = dm.parse_textual_filing(url='https://www.sec.gov/Archives/edgar/data/1318605/000095017022000796/tsla-20211231.htm',return_type='simplify')
```
![Alt text](https://raw.githubusercontent.com/john-friedman/datamule-python/main/static/simplify.png "Optional title")
[Download Example](https://github.com/john-friedman/datamule-python/blob/main/static/appl_simplify.htm)


### interactive html
```
interactive_html = dm.parse_textual_filing(url='https://www.sec.gov/Archives/edgar/data/1318605/000095017022000796/tsla-20211231.htm',return_type='interactive')
```


![Alt text](https://raw.githubusercontent.com/john-friedman/datamule-python/main/static/interactive.png "Optional title")
[Download Example](https://github.com/john-friedman/datamule-python/blob/main/static/appl_interactive.htm)

### json
```
d = dm.parse_textual_filing(url='https://www.sec.gov/Archives/edgar/data/1318605/000095017022000796/tsla-20211231.htm',return_type='json')
```

![Alt text](https://raw.githubusercontent.com/john-friedman/datamule-python/main/static/json.png "Optional title")
[Download Example](https://github.com/john-friedman/datamule-python/blob/main/static/appl_json.json)

## filing viewer
converts parsed filing json into html with features like table of contents sidebar. 

```
from datamule import parse_textual_filing
from datamule.filing_viewer import create_interactive_html
d = parse_textual_filing(url='https://www.sec.gov/Archives/edgar/data/1318605/000095017022000796/tsla-20211231.htm',return_type='json')
create_interactive_html(data,path)
```

future features:
* copy paste and download tables
* copy paste and download text
* download filing


## Known Issues
* Many SEC files are malformed, e.g. this [Tesla Form D HTML 2009](https://www.sec.gov/Archives/edgar/data/1318605/000131860509000004/xslFormDX01/primary_doc.xml) is missing a closing `</meta>` tag among other things.

This error is hard to notice, as if you use a webbrowser it will automatically fix the malformed html for you. In the future, `datamule` will fix SEC errors for you. In the meantime, this should work:
```
from lxml import etree

with open('filings/000131860509000005primary_doc.xml', 'r', encoding='utf-8') as file:
    html = etree.parse(file, etree.HTMLParser())
```

Current solution ideas: detect if html masquerading as xml, parse with lxml etree, and fix file extension.

## TODO
* add mulebot

## Updates
9/18/24
v.302
* re-added output dir to download. Did not mean to remove. Oops.

v0.301
* Fixed package data issue with jupyter notebook
v0.29
* Overhaul. Removes the need to download or construct indices. Expands scope to every SEC filing since 2001, including companies without tickers and individuals. Indexer().watch() has been moved to the downloader. Option to filter by company name has been removed for now due to issues with needing exactly the correct name.

9/16/24

v0.26
* added indexer.watch(interval,cik,form) to monitor when EDGAR updates.
v0.25
* added human_readable option to download, and download_using_api.

9/15/24
* fixed downloading filings overwriting each other due to same name.

9/14/24
* added support for parser API

9/13/24
* added download_datasets
* added option to download indices
* added support for jupyter notebooks

9/9/24
* added download_using_api(self, output_dir, **kwargs). No indices required.

9/8/24
* Added integration with datamule's SEC Router API

9/7/24
* Simplified indices approach
* Switched from pandas to polar. Loading indices now takes under 500 milliseconds.
