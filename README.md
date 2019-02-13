# Belchertown weewx skin

This skin (or theme, or template) is for the [weewx weather software](http://weewx.com) and is modeled after my website [BelchertownWeather.com](https://belchertownweather.com). I originally developed that website with custom coded features but always used weewx as the backend archive software. It was a good fit to remove my customizations and port the site to a weewx skin that anyone can use.

Features include:
* Real-time streaming updates on the front page of the webpage without neededing to reload the website. (weewx-mqtt extension required and an MQTT server with Websockets required)
* Forecast data updated every hour without needing to reload the website. (a free DarkSky API key required)
* Information on your closest Earthquake updated automatically
* Observation charts that update without needing to reload the website.
* Weather records for the current year, and for all time. 
* Responsive design. Mobile and iPad landscape ready! Use your mobile phone or iPad in landscape mode as an additional live console display.
* Progressive webapp ready enabling the "Add to homescreen" option so your website feels like an app on your mobile devices. 

![BelchertownWeather.com Homepage](https://raw.githubusercontent.com/poblabs/weewx-belchertown/master/assets/homepage_screenshot.jpg)

## Table of Contents

- [Belchertown weewx skin](#belchertown-weewx-skin)
  * [Table of Contents](#table-of-contents)
  * [Install weewx-belchertown](#install-weewx-belchertown)
  * [Requirements](#requirements)
    + [weewx.conf](#weewxconf)
    + [DarkSky API (optional)](#darksky-api-optional)
    + [MQTT and MQTT Websockets (optional)](#mqtt-and-mqtt-websockets-optional)
    + [MQTT Brokers](#mqtt-brokers)
      - [Install your own MQTT Broker](#install-your-own-mqtt-broker)
      - [Use a Public Broker](#use-a-public-broker)
  * [Belchertown Skin as Default Skin](#belchertown-skin-as-default-skin)
  * [Using Metric](#using-metric)
  * [Skin Options](#skin-options)
    + [General Options](#general-options)
    + [MQTT Websockets (for Real Time Streaming) Options](#mqtt-websockets-for-real-time-streaming-options)
    + [Forecast Options](#forecast-options)
    + [Earthquake Options](#earthquake-options)
    + [Social Options](#social-options)
  * [Creating About Page and Records Page](#creating-about-page-and-records-page)
  * [Creating a sitemap.xml File](#creating-a-sitemapxml-file)
  * [Add Custom Content to the Front Page](#add-custom-content-to-the-front-page)
  * [Change (or remove) a Chart](#change-or-remove-a-chart)
  * [A Note About Date and Time Formatting in Your Locale](#a-note-about-date-and-time-formatting-in-your-locale)
  * [Frequently Asked Questions](#frequently-asked-questions)
  * [Donate](#donate)
  * [Credits](#credits)

## Install weewx-belchertown

1) Download [the latest release](https://github.com/poblabs/weewx-belchertown/releases).

2) Run the installer as below. Replace `x.x` with the version number that you've downloaded.

```
sudo wee_extension --install weewx-belchertown-x.x.tar.gz
```

3) Edit your `weewx.conf` to [add the required information](https://github.com/poblabs/weewx-belchertown#weewxconf). 

4) Tailor the skin to meet your needs using the [custom option variables. There's a lot of them](https://github.com/poblabs/weewx-belchertown#skin-options).

5) Restart weewx:

```
sudo /etc/init.d/weewx stop
sudo /etc/init.d/weewx start
```

6) Wait for an archive period, or run `sudo wee_reports` to force an update

7) Browse to your website to see the skin. It may be in a belchertown subdirectory.

## Requirements 

### weewx.conf
These settings need to be enabled in order for the skin to work. Within `weewx.conf`, under `[Station]` make sure you have: 
* `latitude` - used for forecasting and earthquake data
* `longitude` - used for forecasting and earthquake data

You need to define your URL for the skin to work properly. The full URL to your website without a trailing slash. Example: `http://yourwebsite.com` or `http://192.168.1.100`.
Even if your website is on your LAN only, this needs to be enabled. There are 2 ways to do this which offer flexibility. See below General Options for more information. You need to only enable 1 option for the skin to work. 
* `belchertown_root_url` - **This is the preferred option**. Add this to your skin Extras section (see below). 
* `station_url` - This is the built-in weewx.conf config. 

### DarkSky API (optional)
DarkSky API is where the forecast data comes from. The skin will work without DarkSky's integration, however it is used to show current weather observations and icons. 

**You must sign up to use their service.** This skin does not provide any forecast data. You need to join their website and get a free developer key. Their free tier allows for 1,000 requests a day. The skin will download and cache every hour - 24 requests a day - well below the free 1,000.

* Sign up at https://darksky.net/dev
* Once you are logged in, take note of the Secret Key on the DarkSky console. 
* Use this key as the `darksky_secret_key` option. See below options table after you have installed the skin.
* Make sure you place the "Powered by DarkSky" somewhere on your website. Like the About page (see below after install for customizing the About page). 

### MQTT and MQTT Websockets (optional)
MQTT is a publish / subscribe system. Mostly used for IoT devices, but it works great for a live website. 

MQTT Websockets allows websites such as this to connect to the MQTT broker to subscribe to a topic and get updates. 

You will need to use an [MQTT broker](https://github.com/poblabs/weewx-belchertown#mqtt-brokers) (aka server) to publish your data to. You can [install your own broker pretty easily](https://github.com/poblabs/weewx-belchertown#install-your-own-mqtt-broker), or use a [public one](https://github.com/poblabs/weewx-belchertown#use-a-public-broker) (some free, some paid). 

Your weewx server will **publish** it's weather data to a broker and visitors to your website will **subscribe** to those updates using MQTT Websockets. When data is published the subscribers get that data immediately. 

With the [`weewx-mqtt` extension](https://github.com/weewx/weewx/wiki/mqtt) installed, everytime weewx generates a LOOP it'll automatically publish that data to MQTT which will update your website in real time. Once ARCHIVE is published, your website will reload the forecast data, earthquake data and graphs automatically.

A sample `weewx-MQTT` extension config is below. Update the `server_url`, `topic`, and `unit_system` to suite your needs. Keep `binding` as archive and loop. Remove the tls section if your broker is not using SSL/TLS.

```
    [[MQTT]]
        server_url = mqtt://username:password@mqtt.hostname:port/
        topic = the/topic/to/publish/to
        unit_system = US
        binding = archive, loop
        aggregation = aggregate
        [[[tls]]]
            tls_version = tlsv1
            ca_certs = /etc/ssl/certs/ca-certificates.crt
```

**I did not write the MQTT extension, so please direct any questions or problems about it to the [user forums](https://groups.google.com/forum/#!forum/weewx-user).**

### MQTT Brokers

#### Install your own MQTT Broker
If you want to run your own MQTT broker, you can [follow these instructions that I've put together](https://obrienlabs.net/how-to-setup-your-own-mqtt-broker/). 

#### Use a Public Broker
These public brokers have been tested as working with MQTT and Websockets. If you have others to add the to the list, let me know.

* [HiveMQ Public Broker](http://www.mqtt-dashboard.com)
* [test.mosquitto.org](http://test.mosquitto.org)
* [You can also try some from this list](https://github.com/mqtt/mqtt.github.io/wiki/public_brokers)

## Belchertown Skin as Default Skin

This is what worked for me to make Belchertown the default skin for your site. This is an **example config** and may need a little fine-tuning site-per-site.

I changed it so the standard skin would be in a subfolder, and the main folder has my skin files. So when you go to my website you're seeing the Belchertown skin, with the default skin under `/weewx`.

1. Edit `weewx.conf`, then look for `[StdReport]` and under it change `HTML_ROOT` to be `/var/www/html/weewx`. Note, your HTML directory may be `/home/weewx/public_html`, so you'd want `/home/weewx/public_html/weewx`.

2. Then modify the Belchertown skin options with these minimal updates. Note, you may need to change the path as mentioned above.

```
    [[Belchertown]]
        HTML_ROOT = /var/www/html
        skin = Belchertown
        [[[Extras]]]
           belchertown_root_url = "http://your_full_website_url"
           
   [[Highcharts_Belchertown]]
        HTML_ROOT = /var/www/html
        skin = Highcharts_Belchertown
```

3. This is optional, but advised: Delete all contents of the `HTML_ROOT` folder and let Belchertown create an entire new site. This prevents stale duplicate data.

4. Restart weewx and let it generate the files upon the next archive interval.

## Using Metric

If your weewx and your weather station are configured for metric, you can display the metric values in the skin. Just like with the [Standard weewx skin](http://weewx.com/docs/customizing.htm#[Units]), to change the site to metric you would need to add `[[[Units]]]` and `[[[[Groups]]]]` to the Belchertown skin options in `weewx.conf`, with the appropriate group values. Restart weewx when you have made the changes. For example:

```
[StdReport]
    [[Belchertown]]
        skin = Belchertown
        HTML_ROOT = belchertown
        [[[Units]]]
            [[[[Groups]]]]
                group_altitude = meter
                group_degree_day = degree_C_day
                group_pressure = mbar
                group_rain = mm
                group_rainrate = mm_per_hour
                group_speed = meter_per_second
                group_speed2 = meter_per_second2
                group_temperature = degree_C
    [[Highcharts_Belchertown]]
        skin = Highcharts_Belchertown
        HTML_ROOT = belchertown
        [[[Units]]]
            [[[[Groups]]]]
                group_altitude = meter
                group_degree_day = degree_C_day
                group_pressure = mbar
                group_rain = mm
                group_rainrate = mm_per_hour
                group_speed = meter_per_second
                group_speed2 = meter_per_second2
                group_temperature = degree_C                
```

## Skin Options

The Belchertown skin will work as a very basic skin once installed using the default values in the table below.

To override a default setting add the setting name and value to the Extras section for the skin by opening `weewx.conf` and look for `[StdReport]`. Under will be `[[Belchertown]]`. Add `[[[Extras]]]` (with 3 brackets) and then add your values. For example:

```
[StdReport]
    [[Belchertown]]
        skin = Belchertown
        HTML_ROOT = belchertown
        [[[Extras]]]
            belchertown_root_url = "https://belchertownweather.com"
            logo_image = "https://belchertownweather.com/images/content/btownwx-logo-slim.png"
            footer_copyright_text = "BelchertownWeather.com"
            forecast_enabled = 1
            darksky_secret_key = "your_key"
            earthquake_enabled = 1
            twitter_enabled = 1
            twitter_owner = PatOBrienPhoto
```

The benefit to adding these values to `weewx.conf` is that they persist after skin upgrades, whereas `skin.conf` could get replaced on skin upgrades. Always have a backup of `weewx.conf` and `skin.conf` just in case! 

Restart weewx once you add your custom options and wait for an archive period to see the results.

For ease of readability I have broken them out into separate tables. However you just add the overrides to the config just like the example above. 

### General Options

| Name | Default | Description
| ---- | ------- | ----------
| belchertown_root_url | "" | The full URL to your website without a trailing slash. Even if your website is on your LAN only, this needs to be enabled. Example: "https://belchertownweather.com" or "http://192.168.0.25/belchertown"
| logo_image | "" | The URL to your logo image. 330 pixels wide by 80 pixels high works best. Anything outside of this would need custom CSS
| site_title | "My Weather Website" | If `logo_image` is not defined, then the `site_title` will be used. Define and change this to what you want your site title to be.
| footer_copyright_text | "My Weather Website" | This is the text to show after the year in the copyright. 
| footer_disclaimer_text | "Never make important decisions based on info from this website." | This is the text in the footer that displays the weather information disclaimer. (available in 0.9)
| manifest_name | "My Weather Website" | Progressive Webapp: This is the name of your site when adding it as an app to your mobile device (available in 0.9)
| manifest_short_name | "MWW" | Progressive Webapp: This is the name of the icon on your mobile device for your website's app (available in 0.9)
| graphs_page_header | "Weather Observation Graphs" | The header text to show on the Graphs page
| reports_page_header | "Weather Observation Reports" | The header text to show on the Reports page
| records_page_header | "Weather Observation Records" | The header text to show on the Records page
| about_page_header | "About This Site" | The header text to show on the About page
| radar_html | A windy.com iFrame | Full HTML Allowed. Recommended size 650 pixels wide by 360 pixels high. This URL will be used as the radar iFrame or image hyperlink. If you are using windy.com for live radar, they have instructions on how to embed their maps. Go to windy.com, click on Weather Radar on the right, then click on embed widget on page. Make sure you use the sizes recommended earier in this description.
| show_apptemp | 0 | If you have [enabled Apparent Temperature](https://github.com/poblabs/weewx-belchertown/wiki/Adding-a-new-observation-type-to-the-WeeWX-database) (appTemp) in your database, you can show it on the site by enabling this. 
| show_windrun | 0 | If you have [enabled Wind Run](https://github.com/poblabs/weewx-belchertown/wiki/Adding-a-new-observation-type-to-the-WeeWX-database) (windRun) in your database, you can show it on the site by enabling this.
| show_cloudbase | 0 | If you have [enabled cloud base](https://github.com/poblabs/weewx-belchertown/wiki/Adding-a-new-observation-type-to-the-WeeWX-database) (cloudbase) in your database, you can show it on the site by enabling this.
| highcharts_enabled | 1 | Show the charts on the website. 1 = enable, 0 = disable.
| highcharts_show_apptemp | 0 | Show the apparent temperature chart on the temperatureplot. Available only on day and week plots.
| highcharts_show_intemp | 0 | Show the indoor temperature chart on the temperatureplot. Available only on day and week plots. (available in 0.9)
| highcharts_show_windchill | 1 | Show the windchill on the temperature plot.
| highcharts_show_heatindex | 1 | Show the heat index on the temperature plot.
| highcharts_graph_1 | "temperatureplot" | Change the observation for chart plot in chart 1. 
| highcharts_graph_2 | "windplot" | Change the observation for chart plot in chart 2. 
| highcharts_graph_3 | "rainplot" | Change the observation for chart plot in chart 3. 
| highcharts_graph_4 | "winddirplot" | Change the observation for chart plot in chart 4. 
| highcharts_graph_5 | "barometerplot" | Change the observation for chart plot in chart 5. 
| highcharts_graph_6 | "radiationplot" | Change the observation for chart plot in chart 6.
| googleAnalyticsId | "" | Enter your Google Analytics ID if you are using one

### MQTT Websockets (for Real Time Streaming) Options

| Name | Default | Description
| ---- | ------- | -----------
| mqtt_enabled | 0 | Set to 1 to enable the real-time streaming website updates from your MQTT Websockets broker (server). In 0.9 use `mqtt_websockets_enabled`.
| mqtt_host | "" | The MQTT broker hostname or IP. In 0.9 use `mqtt_websockets_host`.
| mqtt_port | 8080 | The port of the MQTT broker's **Websockets** port. Check your broker's documentation. In 0.9 use `mqtt_websockets_port`.
| mqtt_ssl | 0 | Set to 1 if your broker is using SSL. In 0.9 use `mqtt_websockets_ssl`.
| mqtt_topic | "" | The topic to subscribe to for your weather data. In 0.9 use `mqtt_websockets_topic`.
| disconnect_live_website_visitor | 1800000 | The number of seconds after a visitor has loaded your page that we disconnect them from the live streaming updates. The idea here is to save your broker from a streaming connection that never ends. Time is in milliseconds. 0 = disabled. 300000 = 5 minutes. 1800000 = 30 minutes

### Forecast Options

| Name | Default | Description
| ---- | ------- | -----------
| forecast_enabled | 0 | 1 = enable, 0 = disable. Enables the forecast data from DarkSky API.
| darksky_secret_key | "" | Your DarkSky secret key
| darksky_units | "auto" | The units to use for the DarkSky forecast. Default of `auto` which automatically selects units based on your geographic location. [Other options](https://darksky.net/dev/docs) are: `us` (imperial), `si` (metric), `ca` (metric except that windSpeed and windGust are in kilometers per hour), `uk2` (metric except that nearestStormDistance and visibility are in miles, and windSpeed and windGust in miles per hour).
| darksky_lang | "en" | Change the language used in the DarkSky forecast. Read the DarkSky API for valid language options.
| forecast_stale | 3540 | The number of seconds before the skin will download a new forecast update. Default is 59 minutes so that on the next archive interval at 60 minutes it will download a new file (based on 5 minute archive intervals (see weewx.conf, archive_interval)). ***WARNING*** 1 hour is recommended. Setting this too low will result in being billed from DarkSky. Use at your own risk of being billed if you set this too low. 3540 seconds = 59 minutes. 3600 seconds = 1 hour. 1800 seconds = 30 minutes. 
| forecast_alert_enabled | 0 | Set to 1 to enable weather alerts that are included with the DarkSky data. If you are using MQTT for automatic page updates, the alerts will appear and disappear as they are refreshed with the DarkSky forecast. 

### Earthquake Options

| Name | Default | Description
| ---- | ------- | -----------
| earthquake_enabled | 0 | 1 = enable, 0 = disable. Show the earthquake data on the front page
| earthquake_maxradiuskm | 1000 | The radius in kilometers from your weewx.conf's latitude and longitude to search for the most recent earthquake.
| earthquake_stale | 10740 | The number of seconds after which the skin will download new earthquake data from USGS. Recommended setting is every 3 hours to be kind to the USGS servers. 10800 seconds = 3 hours. 10740 = 2 hours 59 minutes

### Social Options

These are the options for the social media sharing section at the top right of each page. This does not link your site to anything, instead it gives your visitors a way to spread the word about your page on social media. 

| Name | Default | Description
| ---- | ------- | -----------
| facebook_enabled | 0 | Enable the Facebook Share button
| twitter_enabled | 0 | Enable the Twitter Share button
| twitter_owner | "" | Your Twitter handle which will be mentioned when the share button is pressed
| twitter_hashtags | "weewx #weather" | The hashtags to include in the share button's text. 

## Creating About Page and Records Page

The About Page and Records Page offer some areas for custom HTML to be run. To create or edit these pages, go to the `skins/Belchertown` folder. These files should not be overwritten during skin upgrdades, but it's always best to have a backup just in case!

* Create (or edit) the `skins/Belchertown/about.inc` and `skins/Belchertown/records.inc` files with your text editor, such as Notepad or Nano.
    * These files take full HTML, so you can get fancy if you want. 
    * You can view, and use the sample file [`about.inc.example`](https://github.com/poblabs/weewx-belchertown/blob/master/skins/Belchertown/about.inc.example) and [`records.inc.example`](https://github.com/poblabs/weewx-belchertown/blob/master/skins/Belchertown/records.inc.example). Just rename to remove the `.example`, edit and you should be good to go. 
* Wait for an archive interval for the pages to be generated.

## Creating a sitemap.xml File

Sitemap files are part of the SEO strategy which helps the Search Engine crawlers index your site more efficiently. The result (in addition with other SEO practices) helps visitors find your website through web searches.

Currently with the way that weewx creates websites there is no built-in method which can create a sitemap file automatically. 

Instead this is a manual process. You can use one of the many [online sitemap.xml generator tools](https://www.xml-sitemaps.com) to create one for you. It will crawl your website and create a sitemap.xml file. Download it and place the file into your `HTML_ROOT`  directory.

**Note:** Since the NOAA reports update frequently, you may need to determine a process that works for you to update the sitemap.xml if SEO is important to you. 

You can then submit the full URL to your sitemap to search engine tools. Example:

* [Google](https://www.google.com/webmasters/tools/sitemap-list)
* [Bing](http://www.bing.com/toolbox/webmaster)
* [Yandex](https://webmaster.yandex.com/)
* [Baidu](http://zhanzhang.baidu.com/)

Then insert the following line at the bottom of the `skins/Belchertown/robots.txt` file, specifying the URL path to your sitemap. 

```
Sitemap: http://YOURWEBSITE/sitemap.xml
```

Restart weewx for the changes to robots.txt to update.

## Add Custom Content to the Front Page

There are 4 locations on the front page where you can add your own content. Full HTML is supported. To add content, create a new file in `skins/Belchertown` with the naming convention below. Wait for an archive period for the content to update. 

* Below the station info: `skins/Belchertown/index_hook_after_station_info.inc`
* Below the forecast: `skins/Belchertown/index_hook_after_forecast.inc`
* Below the records snapshot: `skins/Belchertown/index_hook_after_snapshot.inc`
* Below the charts: `skins/belchertown/index_hook_after_charts.inc`

Check out this visual representation:

![Belchertown Skin Custom Content](https://user-images.githubusercontent.com/3484775/49245323-fba5be00-f3df-11e8-982e-dc6363e9f1d1.png)

## Change (or remove) a Chart

You can change the order of your graphs by changing the chart plot name by using the options below in your Extras section. See below in General Options. For example if you wanted chart 6 to be humidity, you set `highcharts_graph_6 = "humidityplot"`. 

If you want to remove a chart, just specify `""`. For example to remove chart 6, you would set `highcharts_graph_6 = ""`. Restart weewx when done.

There are 7 chart plots you can use and they are case sensitive - must be lower case.

* temperatureplot
* windplot
* rainplot
* winddirplot
* barometerplot
* radiationplot
* humidityplot

Here are the default order of the chart plots:

```
    highcharts_graph_1 = "temperatureplot"
    highcharts_graph_2 = "windplot"
    highcharts_graph_3 = "rainplot"
    highcharts_graph_4 = "winddirplot"
    highcharts_graph_5 = "barometerplot"
    highcharts_graph_6 = "radiationplot"
```

## A Note About Date and Time Formatting in Your Locale

In version 0.9 of the skin I decided to move most of the date and time formats to [moment.js](https://momentjs.com/docs/#/parsing/string-format/) using JavaScript. [You can read my thoughts, comments and commits here.](https://github.com/poblabs/weewx-belchertown/issues/56) I feel that moment.js formats the date and time a lot more elegantly than Python. There are so many areas in this skin that use date and time that I've made the decision to let moment.js format these automatically based on your server's locale and timezone. The downside is if you want to change the way it's formatted, you'll need to manually edit the source file to make those updates.

If you notice that there are date, time and timezone formatting that looks wrong for your locale, please set the proper locale and timezone on your weewx server, and restart your server. 

## Frequently Asked Questions

* Q: I don't have a radiation sensor, can I turn that chart off?
* A: You can change the chart to something else, like `humidityplot`. [Check these instructions](https://github.com/poblabs/weewx-belchertown#change-the-charts-order), and restart weewx once you've made the change.
---
* Q: How do I make this skin my default website?
* A: [Click here to take a look at this section of the readme file which explains how to set this up](https://github.com/poblabs/weewx-belchertown#belchertown-skin-as-default-skin). 
---
* Q: My NOAA reports are blank.
* A: If this is right after you installed the skin, give weewx an archive interval (or two, or three...) in order to populate this data
---
* Q: I see errors like these:
    * `No such file or directory '/home/weewx/skins/Belchertown/about.inc'` 
    * `No such file or directory '/home/weewx/skins/Belchertown/records.inc'`
* A: You probably skipped the step [Creating About Page and Records Page](https://github.com/poblabs/weewx-belchertown#creating-about-page-and-records-page). Please give that a try. 
---
* Q: Do I have to use MQTT?
* A: Nope! If you disable the MQTT option, then weewx will still create a website for you, it just will be done on the archive interval. weewx will still generate these pages for you if you have MQTT enabled, the benefit is that you do not have to reload the website. 
---
* Q: What MQTT broker should I use?
* A: [Check the MQTT Brokers section of this page which has more information](https://github.com/poblabs/weewx-belchertown#mqtt-brokers) on a free one that works, as well as **running your own secure broker**. If you want to use a free one, there are a number of them out there and they all have different limitations. Check their terms to make sure it will suite your needs. 
---
* Q: Do I have to use forecasts?
* A: You do not need to use forecasts, but it is recommended to use forecasts so you take advantage of the theme's design with icons and observations.
---
* Q: Do I have to use earthquake data?
* A: Nope! If you leave it disabled, it won't show on the site nor will any data be downloaded. 
---
* Q: Do I have to use the radar?
* A: Nope! If you leave it disabled you'll just have a big blank box on the website. But [windy.com](https://windy.com) provides a free animated radar, so why not include it? :)
---
* Q: Do I have to use the graphs?
* A: Nope! If you have it disabled we will hide those portions of the site. It comes packaged with this theme already though, so you can leave it enabled. 
---
* Q: Why does the skin take a while to generate sometimes?
* A: This is because of the graph system. That file goes through your archive's day, week, month and year values, and all time values to generate the graphs. Depending on how big your database, and how slow your system is (like a Raspberry Pi) is this could take a little longer. If you want to speed it up you can disable the charts or upgrade to better hardware. 
---
* Q: How come the forecast's "Last Updated" time jumps when I load the page?
* A: This is because the page loads with the default Python's format for your locale. When it connects to MQTT websockets, moment.js updates that timestamp to it's format of your locale. This locale format fragmentation is hard to avoid when using locale formatting.
---
* Q: I noticed my graphs don't update right away on an archive period. How come?
* A: Because the highcharts can take a few extra seconds, I've put in a 30 second delay on the graphs automatic update. This way it's loading the newest data.
---
* Q: Do the charts on the Graphs page update automatically with MQTT?
* A: No, only the front page is automatically updated. All the other pages are normal pages that need to be refreshed to see new information.
---
* Q: How do I change my about page, or records page?
* A: [See above on how to do that.](https://github.com/poblabs/weewx-belchertown#creating-about-page-and-records-page)
---
* Q: How can I tell if the skin downloaded new forecast or earthquake data?
* A: Check your system log file. You should see the skin output something along the lines of "New forecast file downloaded" or "New earthquake file downloaded". It will also display errors and what the error was if there was a failure. 
---
* Q: How come I'm seeing `NAN` in some areas?
* A: This is because weewx hasn't gathered enough data from your station yet. Give it a few more archive intervals. 
---
* Q: I'm seeing `cheetahgenerator: **** Reason: could not convert string to float: N/A`, how do I fix this?
* A: Upgrade to 0.8.1 or newer which resolves this error
---
* Q: How do I uninstall this skin?
* A: `sudo wee_extension --uninstall Belchertown`

## Donate
[![Donate](https://img.shields.io/badge/Donate-Support%20by%20Donating%20or%20Buying%20me%20a%20Coffee-blue.svg)](https://obrienlabs.net/donate)

This project took a lot of coffee to create. If you enjoy this skin and find some value from it, [click here to buy me another cup of coffee](https://obrienlabs.net/donate) :)

## Credits
* DarkSky API for the weather forecasts.
* Windy.com for the iFrame embedded weather radar.
* Gary for the [weewx-highcharts](https://github.com/gjr80/weewx-highcharts) extension. A custom forked version has been packaged with this Belchertown theme. 
* Brian at [weather34.com](http://weather34.com) for the weather icons from the simplicty 2015 theme. Used with agreement.
