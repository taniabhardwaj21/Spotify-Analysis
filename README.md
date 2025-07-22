# Spotify Listening Trends Analysis


## 📌Problem Statement

In the modern era of music streaming, understanding user listening behavior is critical for platforms like Spotify. By analyzing detailed user playback data, we can uncover insights such as preferred listening times, popular albums/artists, and behavioral patterns (e.g., skipping or shuffling tracks). This project aims to answer:

"How do user listening habits evolve over time, across artists, albums, and tracks, and how can these insights inform personalization and user engagement strategies?"

## 📊 Project Overview

This Power BI project explores a Spotify streaming history dataset to uncover trends in music consumption. The dataset includes:  track_name, platform, ms_played, artist_name, album_name, shuffle, skipped, track_played_date, track_played_time

The analysis is broken into three dynamic dashboards, each addressing a unique aspect of listening behavior.

## 🛠 Tools & Technologies
Power BI Desktop

DAX (Data Analysis Expressions)

Data Modeling

Data Visualization & UX

CSV Export and Drillthrough

## 📦 Project Highlights
✅ Real-world use case for user behavior analytics

✅ End-to-end Power BI workflow (load → clean → model → visualize)

✅ Fully interactive dashboards with filtering, drillthrough, and slicing

✅ Robust DAX use for advanced calculations and comparisons



## 📁 Dataset Processing Steps

### 1. Loading the Data
Imported the Spotify listening history into Power BI from CSV format.

### 2. Data Cleaning
Separated the ts column into track_played_date and track_played_time.

Created a new Year column by extracting year from the track_played_date.

DAX - 
          
            Year = YEAR('spotify_history'[track_played_date])

### 3. Date Table Creation
Created a dynamic date table using the minimum and maximum dates from the spotify_history table. DAX -

        Date Table = CALENDAR(MIN(spotify_history[track_played_date]), MAX(spotify_history[track_played_date]))

Extracted year from this date table:

DAX-

        Year = YEAR('Date Table'[Date])

### 4. Data Modeling
Established relationships between the date table and the main data table based on the date fields.

### 5. Canvas Customization
Configured the report background and theme for a professional, clean visual presentation.
        
## 📘 Dashboard 1: Albums, Artists, and Tracks Analysis
### ALBUMS
#### ✅ Total Albums Played Over Time

Calculated distinct count of albums played over time:

DAX-

        No. of Albums = DISTINCTCOUNT(spotify_history[album_name])

#### ✅ Number of Albums Listened by Year (with Min/Max Highlight)

Visualized as area chart over time. Added a measure to show only Min and Max values:

DAX- 

        MinMax Albums Line chart = 
        VAR _MinValue = MINX(ALLSELECTED('Date  Table'[Year]), CALCULATE(DISTINCTCOUNT(spotify_history[album_name])))
        VAR _MaxValue = MAXX(ALLSELECTED('Date Table'[Year]), CALCULATE(DISTINCTCOUNT(spotify_history[album_name])))
        VAR _CurrValue = DISTINCTCOUNT(spotify_history[album_name])
        RETURN 
        IF(_CurrValue = _MaxValue || _CurrValue = _MinValue, _CurrValue, BLANK())


#### ✅ Weekday vs Weekend Album Listening Patterns
Created two new columns:

DAX- 

        Day Name = FORMAT('Date Table'[Date], "DDD")

        Weekday_Weekend = IF(WEEKDAY('Date Table'[Date], 2) <= 5, "Weekday", "Weekend")
Visualized using a donut chart.

#### ✅ Top 5 Albums
Used a clustered bar chart with top N filtering on album_name based on frequency.

#### ✅ Year-over-Year Album Comparison
Latest Year Albums:

DAX-

        Latest Year Albums = 
        VAR _latestyear = 
            CALCULATE(
                MAX('Date Table'[Year]), 'Date Table',
                TREATAS(VALUES(spotify_history[Year]), 'Date Table'[Year]),
                VALUES(spotify_history[platform])
            )
        RETURN 
            CALCULATE(DISTINCTCOUNT(spotify_history[album_name]), 'Date Table'[Year] = _latestyear)
Previous Year Albums:

DAX - 

    Prev Year Albums = 
        VAR _latestyear = 
            CALCULATE(
                MAX('Date Table'[Year]), 'Date Table',
                TREATAS(VALUES(spotify_history[Year]), 'Date Table'[Year]),
                VALUES(spotify_history[platform])
            )
    RETURN
        CALCULATE(DISTINCTCOUNT(spotify_history[album_name]), 'Date Table'[Year] = _latestyear - 1)
Year-on-Year Comparison:

DAX -

    PY and YoY = 
    VAR latest = [Latest Year Albums]
    VAR prev = [Prev Year Albums]
    VAR YoY = IF(NOT(ISBLANK(prev)), DIVIDE(latest - prev, prev, 0), BLANK())
    RETURN
        IF(NOT(ISBLANK(prev)), 
        "vs PL: " & FORMAT(prev, "#,##0") & " (" & FORMAT(YoY, "0.00%") & ")", "No Data")

### ARTISTS
Applied the same techniques and visuals as Albums:

- No. of Artists = DISTINCTCOUNT(spotify_history[artist_name])
- Min/Max logic reused
- Donut chart for weekday/weekend
- Top 5 Artists chart
- LY, PY, and YoY measures replicated with artist_name

### TRACKS
Similar structure:

- No. of Tracks = DISTINCTCOUNT(spotify_history[track_name])
- Min/Max logic reused
- Donut chart for weekday/weekend
- Top 5 Tracks chart
- LY, PY, and YoY measures replicated with track_name

### Additional Features:
Slicers for:
- platform
- shuffle (Yes/No)
- skipped (Yes/No)

Edited visual interactions so only intended visuals respond to filters

## 📘 Dashboard 2: Listening Patterns
### ✅ Heatmap: Listening Hours by Day
Created a matrix:

- Rows: Hour
- Columns: Day Name
- Values: COUNT(track_name)

Created helper column to sort weekday names:

DAX- 

    Day Number = WEEKDAY('Date Table'[Date], 2)
Applied conditional formatting on both the heatmap and bar chart showing track counts by hour.

### ✅ Scatter Plot: Avg Listening Time vs Track Frequency
#### Average Listening Time (min):

DAX-

    Avg Listening Time (min) = AVERAGE(spotify_history[ms_played]) / 60000
#### Track Frequency:

DAX-

    Track Frequency = COUNT(spotify_history[track_name])
Used these to create scatter plot. 

Introduced two parameters for quadrant thresholds:

- Listening Time (min) parameter
- Track Frequency (parameter) parameter

Dynamic quadrant coloring using:

DAX- 

    CF Quadrant = 
    VAR Avgtime = [Avg Listening Time (min)] <= 'Listening Time (min)'[Listening Time (min) Value]
    VAR TrackFreq = [Track Frequency] >= 'Track Frequency (parameter)'[Track Frequency (parameter) Value]

    VAR Result = SWITCH(
        TRUE(),
        Avgtime && TrackFreq, 1,
        NOT Avgtime && TrackFreq, 2,
        NOT Avgtime && NOT TrackFreq, 3,
        Avgtime && NOT TrackFreq, 4
        )
    RETURN Result
Applied this measure to marker color, highlighting only quadrant 2 (engaging tracks).

## 📘 Dashboard 3: Grid View with Drillthrough
Created matrix grid with Album Name → Artist Name → Track Name hierarchy.

Enabled:

- Drill Down & Up
- Drillthrough navigation from main dashboards
- CSV Export of drilled-in data

### 🔍 Insights Derived

-   No. of Albums, artists and tracks played increase tremendously from 2019 to 2021. After 2021 there is a gradual decrease in streaming.
- Most of the songs are played on Android, with other platforms seeing glaringly low playing

- Weekend peaks and evening hours dominate listening trends with morning hours on weekdays having the least songs played. The most played hour is midnight while the least is noon.

- Repeat listening concentrated on top 5 tracks/artists.

Track skipping and shuffling vary by day and platform.

YoY analysis shows increased diversity in listening behavior.

# Snapshot of Dashboards

![Image](https://github.com/user-attachments/assets/a8c5eb7d-d6ab-40e1-88e4-1fd777d4eba2)

![Image](https://github.com/user-attachments/assets/14b9d6a1-26b7-4541-92d4-3b08b41cda7e)

![Image](https://github.com/user-attachments/assets/ab60c2b3-4be8-4b1f-a041-668fd4b4ea3f)
