NOT COMPLETE, STILL WIP

# Project Brief: Nutrition, Physical activity, and Obesity, Behavioral Risk Factor Surveillance System

This dataset comes from the [CDC](https://data.cdc.gov/Nutrition-Physical-Activity-and-Obesity/Nutrition-Physical-Activity-and-Obesity-Behavioral/hn4x-zwk7/about_data), and it includes data on adult's diet, physical activity, and weight status from Behavioral Risk Factor Surveillance System.

With this dataset, I wanted to look at one question that came to mind while I was looking at the data, How does obesity, nutrition, and physical activity differ across states and demographic over time?
Some of the sub-questions that come with this are:
	- Trend of the three factors over time
	- State comparison of the three factors
	- Demographic(education, income) distribution relative to the three factors

Looking at these questions is important to understanding the differing levels of health factors across states, some of these health factors can point to the disparity of income across states, and not within these but left for speculation and further testing is: levels of access to not only nutritional-based needs but also physical activity needs like access to a gym or a park and overall behavioral health as these factors can all have some impact on that. 

The audience for this is a general portfolio viewer, this is a mix of showcasing my and practicing my skills within PowerBI as well as producing genuinely informative points on a dataset that comes from a health organization.

## Data Exploration, Cleaning & Shaping

While in Power Query, I wanted to look at which columns out of the 33 available, are most important to me in answering the questions I have. Identified, here are the columns: YearStart, LocationDesc, Topic, Question, Data_Value, Data_Value_Footnote(for identifying nulls), Sample_Size, Education, Income, and StratificationCategory1.
 
Now, for deleting these rows, there isn't a particular cluster where null data exists mainly, not in Years, Location, or Question, all of the nulls are evenly scattered and nothing suggests one factor mainly produces nulls. Since this is mainly just random noise, it is completely fine dropping these rows as assuming nulls are evenly scattered allows the data to be deleted and have nothing be affected or one thing becomes underrepresented while others are overrepresented.

The total number of nulls I will be deleting is 13,214 which is about 11.9% of the data.
	- 13,214/110,880

After removing the unwanted columns, I was left with the ones I listed above and I checked all data types to ensure everything lined up with what would make sense, 123 for numeric values, abc for text.

The values in YearEnd are the same as YearStart for all rows, so this indicates that YearEnd isn't really needed, and this also assumes that YearEnd is the same as YearStart, I had to check before I completely got rid of it.
[YearStart] <> [YearEnd] returned only False, this shows that all of the values in YearEnd are the same as in YearStart, so it is safe to remove that column.

With all of those in check, I applied the changes and will start with modeling my data.

## Data Modeling

I opted for a single flat table rather than splitting into multiple related tables because the dataset's size does not approach the scale where splitting the data would meaningfully support performance.

In order to get to the answer of looking at a single trend over time, I have to create interactive measures that allow me to look at a single trend over time. To do this, I took it in three steps:

1) Create a current average measure, this is using the DAX formula, AVERAGE() which allows me to get an average of a collection of items. I do not really end up with a collection of items, it is just 1 item that, when filtered by the slicer, I have just 1 item that becomes the "average".

2) Prior Year Avg which utilizes the CALCULATE() DAX formula and that takes my Current Avg measures created earlier and to get the prior year I do the MAX() of YearStart - 1, subtracting by 1 gives the prior year. The MAX() formula is used to get a singular value out of the YearStart column, this column can return duplicate years and to get just one, MAX() or even MIN() would work to get us to that objective. And CALCULATE() is used here because of twofold: it overrides YearStart and it leaves Question and StratificationCategory1 untouched because these measures are user selected and should only be changed by them.

3) To combine all three, I then create a Change vs Prior Year measure that subtracts my current year by the prior year, and that gives me the change for the two selected years

To test these measures I created a Card visual with Year/StratificationCategory1/Question and confirmed the values against my data to ensure the measures were correct.

## Visual Design & Layout

Trend over time: Considered having one line chart that combines all three topics into one, but this would increase the complexity of understanding for the viewer. Having multiple distinct questions(Obesity: 2, Fruits & Vegetables: 2, Physical Activity: 5) within each topic would also make this harder to visualize and understand, there was no single "question per topic" line within the chart that would capture the full picture. Since the topics were also measured differently, there was also no axis that would make for a decent comparable scale, you'd be losing out on important, meaningful data if you were to try to squeeze all topics within the same scale.
	- The resolution for this was to create three separate line-charts, one per topic, with each line being a sub-question(using the Current Avg measure across YearStart, filtered to StratificationCategory == "Total" to avoid mixing demographic breakdowns, which I plot later.
	- For state comparisons, I followed the same one visual per topic pattern as the line-chart where each topic gets it's own map, with the ability to click through the sub-questions, filtering the map to the chosen sub-question, I did this using a slicer.
	- For demographic distribution: I added two clustered bar charts per topic page, broken out by Income and Education, and filtered StratificationCategory1 to the respective "Income" and "Education".
	- Slicer linkage: For obesity, the slicer is linked to the map and the two clustered bar charts, this is done so that the viewer can filter to the sub-question they would like to view; this does not apply for the line chart because it accurately displays all of the lines and the sub-questions without causing messy visualizations. For Physical Activity, not all of the sub-question had data for the years past 2022, so I kept the map at the "Percent of adults who engage in no leisure-time physical activity" for the year 2024, because that is the most recent year and there is data for it. I did this by creating a YearStart filter for 2024 set on only the Map visual because I did not want it to impact the other visuals. The slicer for this topic works on the clustered bar charts since it still produces data for the sub-questions within those charts.
		- I decided not to use the slicer on the map where I find the most recent years with data for each of the sub-questions because I felt that would create too much inconsistencies across the 4 tabs and would confuse the viewer. Especially with Physical Activity having 5 different sub-questions, finding distinct years within each sub-question might get too confusing. 

The Overview page has a map of the US showing the most interesting stat within this dataset, the rising rate of obesity year-over-year change across states since 2011 and how many points it has risen since. While the other two topics provided meaningful and significant data, the Obesity rates were the easiest to hook the viewer.

Missing Data Handling: Tennessee and Florida were both missing data in one of the tabs and to handle this I added a small annotation documenting it.
