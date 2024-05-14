# Understanding Demographic Changes in the United States

## Project Overview

We started this project with the goal of understanding demographic changes in the United States. Given the large number of cities, we decided to focus on counties to provide a manageable and insightful level of granularity. Counties strike a balance between the broad overview provided by states and the plethora of details provided by cities, making them ideal for our analysis.

## Visual Coding

For our visual coding, we chose a choropleth map to represent the population data. This choice allows us to effectively use color intensity to convey the population density of different regions. By using the blue scheme, we can clearly distinguish between counties with different populations. Dark colors indicate higher population, while light colors indicate lower population. Counties with missing data are shown in black to ensure clarity and accessibility.

## Data Preprocessing

In the data preprocessing stage, the estimated population is calculated by adding the population base with births, subtracting deaths, and adding migration. Since estimated populations are large numbers, we applied a natural logarithm (log base e) to scale the estimated population shown in our visualization, making it easier to see differences among different areas.

## User Interaction Enhancements

To enhance user interaction, we added the following features to the map:

1. **Zooming and Tooltips**: Users can zoom in and out of the map, and when the mouse hovers over an area, the number of people in the county is displayed, along with the name of the county and the name of the state. This interactive element encourages users to explore the data in greater detail.
2. **Search Feature**: Users can search for a specific county by entering the "county, state" order. When a search is performed, the map automatically zooms in and highlights the county in red. Hovering over the highlighted county displays the number of people in that county, providing immediate visual feedback. After canceling the search, the map reverts to its original size.
3. **Click to Zoom**: Users can click on a county area on the map to zoom into that area. When the mouse hovers over the area, demographic information is displayed. Clicking the same area again will revert the map to its original size.
4. **Time Slider**: We track the change of estimated population from 2020 to 2023. A slider allows users to visualize changes in the population across the whole map over time. By moving the slider, users can see how the population of each county has changed year by year.

## Considered Alternatives

We also considered other visualization methods such as heat maps and bar charts. However, heat maps are less effective at showing discrete regions such as counties and states, and bar charts lack the geographic context we need for our analysis. Ultimately, the interactive terrain map we chose best balances clarity, usability, and detailed geographic representation.

## Development Process

Our development team consisted of three members:

- **Yuxuan Zhang**: Further refined the data cleaning, made it suitable for use on the website, and created the structure of the website. Yuxuan integrated data with visual components, laying the foundation for our project. (16 hours)
- **Tony**: Responsible for data mining and initial data cleaning, ensuring the data was accurate, and displaying our results on GitHub. (14 hours)
- **Yiming Jia**: Involved in the creation of the website and added interactive tools to ensure that the website is visually appealing and user-friendly. (14 hours)

Throughout the development process, the most time-consuming tasks were data cleaning and ensuring the display of the visualizations.
