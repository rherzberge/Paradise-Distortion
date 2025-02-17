# Daijishō Search Engine Scraper Syntax (DSESS)

DSESS uses search engine URL to obtain target sites. Then use the CSS selector syntax to acquire wanted sources from target sites.

## Preknowledges
### CSS selector
Before further instruction. Make sure to view the [doc](https://jsoup.org/cookbook/extracting-data/selector-syntax) from Jsoup.

### URL Query string
Query string is a part of a URL that assigns values to specified parameters. Which is useful for search engine.
Your parameters have to be encoded as [URL Query encoding](https://www.url-encode-decode.com/).

### Regex
See https://regex101.com/.

## Overview
DSESS has below format.

`Headers` + `:` + `Template tags` + `:` + `DSESS URL`

## Headers
`DSESS:` + scraper target

`DSESS:BOX_ART`

`DSESS:SNAPSHOT`

`DSESS:TITLE`

`DSESS:YOUTUBE`

`DSESS:DESCRIPTION`

## Template tags
You have to specify template tags you want to use inside the `TAGS()` brackets. Separated by comma without spaces. It is case sensitive. For example:

`TAGS(scraperKeyword,platformName)`

DSESS string with more tags are recommended to order it at a higher priority position in your platform JSON file. Since it is more specific. However if Daijishō can't provide those tags, that scraper string will be ignored.

In order to use template tags in query. Surround it with curly brackets in query. Like:

`PlayStationPortable {scraperKeyword}` `q={scraperKeyword}&hl={localeLanguage}` `query={scraperKeyword}`

Encoded as

`PlayStationPortable+%7BscraperKeyword%7D` `q=%7BscraperKeyword%7D&hl=%7BlocaleLanguage%7D` `query=%7BscraperKeyword%7D`

### Template tags
- scraperKeyword
- platformName, like `DOOM Game Engine`
- localeLanguage, like `en` `zh`
- localeCountry, like `US` `TW`
- localeLanguageAndCountry, like `en-US` `zh-TW`

## DSESS URL
DSESS URL contains **The body URL** and several **DSESS URL parameters**.
DSESS defined parameters will be extracted and removed before HTTPS request.

### 0. The body URL
The body URL contains the search engine HTTPS URL with template tags. The template tags will apply to each URL parameter when each paramenter be obtained. For example:

`https://www.google.com/search?q=%7BscraperKeyword%7D&hl=%7BlocaleLanguage%7D&tbm=isch`

`https://www.gametdb.com/`


You can see  `{scraperKeyword}` is encoded as `%7BscraperKeyword%7D`. So do `{localeLanguage}`. And those will be replaced by corresponding string values.

All parameters **must be encode to URL query encoded string**. Concatenate by URL parameters rules.
Same with following DSESS URL parameters. Also the DSESS URL parameters are processed with below order.

### 1. Target site link selector parameter
`dsess_target_site_selector=` + Target site `<a>` element CSS selector.

For example: `div.col-xs-9.col-sm-9.col-md-9.col-lg-9 a`.

And don't forget to **encode the Regex to URL encoded query string** like others parameters.

Encoded as: `div.col-xs-9.col-sm-9.col-md-9.col-lg-9+a`.

It will matches all `<a>` elements available from search results from search engine. These `<a>` elements will be used by the next "**2. Target site Regex**" in next process.

If `dsess_target_site_selector` is not present. All `<a>` element will be used.


### 2. Target site Regex parameter
`dsess_target_site=` + Target site Regex.

For example: `^https:\/\/www.romspedia.com\/roms\/.*$`.

And don't forget to **encode the Regex to URL encoded query string** like others parameters.

Encoded as: `%5Ehttps%3A%5C%2F%5C%2Fwww.romspedia.com%5C%2Froms%5C%2F.%2A%24`.

It will matches all links filtered by **1. Target site link selector** from search engine. And if matched, the first matched by "**2. Target site Regex**"  will be used by "**2.selector parameter**" in next process.

If `dsess_target_site` is not present. The search engine site itself will be used by **3. Selector parameter**'s CSS selector.


### 3. Selector parameter
`dsess_selector=` + CSS selector.

When target site is available, the selector parameter will be applied on target site.


### 4. Attribute parameter
`dsess_attribute=` + Attribute you want from the element selected by CSS selector.

For example `href` for `<a>` element.
If `dsess_attribute` is not present the doc's html() will be resolved.


### 5. Extractor Regex parameter
`dsess_extractor=` + Extractor Regex.

If `dsess_extractor` is not present the the plain text will be used. Else string extracted from Regex group 1 will be used.


### 6. Replacer Regex parameter
`dsess_replacer=` + Replacer Regex.

If `dsess_replacer` is not present the the original text will be used. Else string filter by it will be used in **7. Replacer value parameter**.

### 7. Replacer value parameter
`dsess_replacer_value=` + Replacer value.

If `dsess_replacer` is not present the the original text will be used. Else string filter by **6. Replacer Regex parameter** will be replaced with the replacer value.

## Example
`DSESS:BOX_ART:TAGS(scraperKeyword):https://www.switchscores.com/games/search?search_keywords=%7BscraperKeyword%7D&dsess_target_site_selector=div.col-xs-9.col-sm-9.col-md-9.col-lg-9+a&dsess_target_site=%5Ehttps%3A%5C%2F%5C%2Fwww%5C.switchscores%5C.com%5C%2Fgames%5C%2F.%2A%24&dsess_selector=div.col-md-8+%3E+img.img-responsive&dsess_attribute=src`

`Box art` for nintendo switch games on www.switchscores.com.

### Decoded parts

`The body URL` `https://www.switchscores.com/games/search?search_keywords={scraperKeyword}`

`dsess_target_site_selector=` `div.col-xs-9.col-sm-9.col-md-9.col-lg-9 a`

`dsess_target_site=` `^https:\/\/www\.switchscores\.com\/games\/.*$`

`dsess_selector=` `div.col-md-8 > img.img-responsive`

`dsess_attribute=` `src`

`dsess_extractor` is not used in this example
