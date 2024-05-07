By Gabe Haddad and Breitling Snyder

<h3><span style="color:#434343;font-size:13.999999999999998pt;">Categorization of Census blocks in the East Bay based on their mobility access using Python analytical and mapping tools.</span></h3>



<h1><span style="font-size:20pt;">Introduction:</span></h1>
The purpose of this project is to visualize access to transportation across the East Bay. Our goal was to identify which areas have the highest and lowest measures of access to non-auto mobility. Access to urban mobility is a key socioeconomic metric of neighborhoods in large cities. Our intent was to evaluate the spatial equity of the transportation network across our area of analysis. 

To do so, we developed a series of indices that rank each Census Block in Berkeley, Oakland, and Emeryville based on their proximity to transportation facilities and the density of the transportation network. We first calculated access to nearby bus stops, BART stations, and Bay Wheels bike share docks along the walk network. Each of these modes serves a different need. BART is a regional rail network that can provide longer distance trips across the Bay Area region. These trips across the region can be in excess of 30 miles. AC Transit serves municipalities within Alameda County and has a far denser network of stations than BART. It is the primary public transit network for short to medium-distance trips within the East Bay zone we analyzed. Bay Wheels is a privately provided bike share service that is best suited for short distance trips as a substitute for walking. We then factored in the density of the pedestrian and bike infrastructure within the vicinity of each Block. By factoring in proximity to rail, bus, bike share, and active transportation infrastructure, our index paints a comprehensive picture of the mobility level of each Census Block in the region.

We also wanted to see how our own mobility ranking compares to government initiatives to promote spatial equity. Access to active transportation facilities, as well as short, medium, and long distance transit options is crucial in megaregions like the Bay Area. Land use patterns and high costs of driving in the region necessitate access to both local and regional transit, especially for low-income individuals and families. To evaluate how the transit and active transportation networks serve low-income communities, we mapped the locations of Low Income Housing Tax Credit (LIHTC) affordable housing properties. Overlaying these locations on top of our mobility index helps us understand the level of mobility access that affordable housing residents have. We selected the LIHTC program because it is by far the largest affordable housing program both nationally and in the Bay Area. LIHTC properties account for over 75% of overall affordable housing units in the East Bay, and have the most available data for spatial analysis. We plotted the location of these properties on top of our mobility index to paint a visual picture of where they are relative to high and low mobility areas of the region and calculated the average scores of the Census Blocks that each property sits on broken out by each of the three cities we analyzed. 

Overall, we found that Berkeley is slightly more accessible than Oakland and Emeryville. We also found that LIHTC properties are, on average, located in high-mobility areas, which is an indicator that these residents have good access to transportation relative to the overall level of service in their cities. Areas of lower accessibility include East Oakland and the Oakland Hills. 



<h1><span style="font-size:20pt;">Methodology:</span></h1>

We chose Census Blocks as our unit of spatial analysis because we wanted the most granular zones available without being too computationally heavy. Given the density of non-auto transportation facilities across the area of analysis, Blocks that are next to each other can have meaningfully different mobility access. In addition, as we only dealt with built environment characteristics, we did not have to worry about data not being available at this small of a spatial level. We considered using individual properties to maximize the precision of our index, but the number of calculations required was too high. 

For our area of research interest, we chose to examine the East Bay, which we defined to be the cities of Oakland, Piedmont, Emeryville, and Berkeley. Within our study area, there were 6890 Census Blocks for which we did each of our calculations. 

We calculated and included the following variables in our index:

Routing Variables:
<ul>
  <li>Distance to the closest BART station along the walking network</li>
  <li>Distance to the closest bus stop along the walking network</li>
  <li>Distance to the closest BayWheels bike share station</li>
</ul> 
             
Network Length Variables:
<ul>
 <li>Ratio of the length of the walk to road network</li>
 <li>Ratio of the length of the bike to road network</li>
</ul> 
       
<h2><span style="font-size:16pt;">Data Collection and Wrangling </span></h2>

The following table describes all datasets used and their sources. It can be assumed that all data used was the most recent data provided by these sources.

<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/785d6d30-645b-4f82-87ab-0d265050b513" width="75%" />
</p>


Once we clipped the Census Blocks to the research area of interest, we removed any Blocks that contained only water and then found the centroid of each. All datasets were stored as GeoDataFrames with CRS set to World Geodetic System 1984 (EPSG 4326). 


Figure 1: Map of BART stations, AC Transit stops, and Bay Wheels ports
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/7107797e-e320-4c11-8a62-b81d5304b305" title="East Bay Mobility Assets">
</p>


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Routing Variables</span></h3>
To rank access to transportation, we calculated the distance from the centroid of each Block to the nearest BART station, bus station, and Bay Wheels port. We calculated the distance along the walk network, as we assumed (or hoped) that most people would be walking to these transportation facilities rather than driving. We created a walk network for the East Bay region using OSMnx, which used the OverPass API to access OpenStreetMap with a predefined query for walkable facilities. Though we used OSMnx to define our network, it was much more computationally heavy to use for routing calculations, and therefore we converted the OSMnx network to a Pandana type network to do routing with this package instead. 

Because the Block centroids and transportation facility locations are typically not already located on the network, we used the Pandana get_node_id function to snap each origin (Block centroid) and destination (transportation facility) point to the network by adding this node ID information to our datasets. For each of our three routing variables, we then iterated through Census Blocks to find the distance from each origin to all possible destinations and selected the minimum distance among these. Even using Pandana this proved to be computationally complex, with over 15 million total routing calculations for the distance to bus stop variable alone. These minimum network distance variables were added to our Census Block dataset. 


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Network Length Variables</span></h3>
To score the walk and bike networks for a given Census Block, we calculated the ratio of walk and bike networks to the road network, respectively. We assume that higher ratios mean more options for active transportation routes relative to the driving network. For instance, a walk-to-drive ratio of 2.0 means that there are twice as many opportunities in an area to walk as opposed to drive. 

Figure 2: Maps of Walk, Bike, and Drive networks
<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/d399e349-06d0-4db4-90b2-f60ca7d7f10a" width="32%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/99b1ecf4-180c-48cd-b88b-6032079c5a1f" width="34%" /> 
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/1284d208-50ce-4d4e-be5f-1a82955444f4" width="32%" />
</p>

One challenge that we ran into when developing these metrics was the scale of the Census Block. Blocks are typically quite small, especially in more densely populated areas like the East Bay, and their boundaries are typically defined along the existing road network– therefore it is somewhat up to chance whether a road falls in one Census Block versus another. In addition, people’s experience of their active transportation options is typically not limited to just what is outside of their front door. To expand the scale of our network analysis, we chose to define a function that would create a larger polygon on which to create our networks. This polygon was defined by all of the Census Blocks whose centroids fell within a half-mile radius of the original Block of interest’s centroid. Using this expanded scale, we could gain a better understanding of the active transportation options within a given area. 

We then iterated through Census Blocks and created walk, bike, and road networks on our expanded scale using OSMnx-defined queries for these network types, as done previously for the routing variables. We chose to retain all pieces of each network even if nodes were not fully connected, as we were only interested in the total length of the network and did not have to route between potentially unconnected nodes. Once the length of all edges in the networks were calculated using the OSMnx basic_stats function, we divided the walk and bike network lengths by the road network length, respectively, and added these values to our Census Block dataset.

<h2><span style="font-size:16pt;">Variable Normalization</span></h2>
<p><span style="font-size:11pt;">In order to add all of our variables to the index and assign accurate weights to each, we first normalized each variable. For our routing variables, larger values indicated lower accessibility, so therefore we inverted the distribution and normalized using the following equation:</span></p>
<p style="text-align: center;"><span style="font-size:11pt;">X</span><span style="font-size:11pt;"><sub><span style="font-size:0.6em;">norm&nbsp;</span></sub></span><span style="font-size:11pt;">= (X</span><span style="font-size:11pt;"><sub><span style="font-size:0.6em;">max</span></sub></span><span style="font-size:11pt;">&nbsp;- X) / X</span><span style="font-size:11pt;"><sub><span style="font-size:0.6em;">max</span></sub></span></p>
<p><span style="font-size:11pt;">For our network length variables we did a relatively simpler normalization, since higher values meant better opportunities for active transportation. We normalized these according to the following equation:</span></p>
<p style="text-align: center;"><span style="font-size:11pt;">X</span><span style="font-size:11pt;"><sub><span style="font-size:0.6em;">norm&nbsp;</span></sub></span><span style="font-size:11pt;">= X / X</span><span style="font-size:11pt;"><sub><span style="font-size:0.6em;">max</span></sub></span></p>

<h2><span style="font-size:16pt;">Developing Indices</span></h2>

We developed three indices to evaluate mobility access based on the five metrics (distance to BART, AC Transit, and Bay Wheels, plus bike and walking network density) we gathered from our spatial data. Our indices differ based on how much weight is placed on each metric. Our indices are based on our personal logical assumptions rather than a quantitative criteria. We acknowledge that others may weigh the modes differently. Our code permits us to tweak the weighting to emphasize certain modes. Our weighting reflects our opinions on how much each metric impacts the overall mobility level of each Census Block.

Our first index is oriented toward transit options rather than infrastructure, so we weighted rail, bus, and bike share but left the multiplier for walk and bike network at 1x. Distance to BART was weighted at 8x because the rail network’s coverage is far greater than bus or bike share which has meaningful implications for residents whose jobs or families may live in other regions within the Bay Area that cannot be reached by bus, let alone bike. We weighted AC Transit at 4x because its coverage is far less than that of BART’s, but its network is much denser and can access much more specific locations within the East Bay. Bay Wheels was given a 2x multiplier because it roughly doubles the radius an able-bodied person would be willing to walk. Our analysis was conducted before electric bikes were launched in the East Bay.

Our second index is focused on bike and walking networks. Distances to the three transit modes are weighted at 1x. The walk network density is given a 4x multiplier and the bike network density is given an 8x multiplier. We weighted the bike network at twice the value of walking because the presence of bike lanes drastically increases safety and therefore the likelihood that one will bike as a mode of practical transportation. The walk network is weighted less because jobs, services, and amenities are often beyond walking distance from one’s residence, but the walk network does play a large role in a neighborhood’s internal accessibility for residents.

Our third and most important index combines all five mobility metrics and is what we called the “combination score.” BART, AC Transit, and Bay Wheels have the same weights as the transit index at 8x, 4x, and 2x respectively. The logic outlined for the transit score carries over to the combination score. The walk network density was scored at 1x under the assumption that walking is the baseline ability to move throughout a region. Bike network density is scored at 3x with the assumption that realistic trip distance is at least 3x that of walking, and that the presence of bike infrastructure greatly increases the safety and likelihood of using biking as a mode of transportation.

<h2><span style="font-size:16pt;">LIHTC Comparison</span></h2>

For the LIHTC properties, we plotted their locations as points on top of the combination score map to see where they are placed relative to overall mobility levels. We then calculated the average mobility score for all Blocks within the cities of Berkeley, Oakland, and Emeryville that contain LIHTC properties. We did this to evaluate how affordable housing mobility access compares across the three cities we analyzed as well as to the mobility levels across the full area of analysis. We found an average combination score for LIHTC locations in each city and compared them to the distribution of scores we calculated for the full East Bay region.

<h1><span style="font-size:20pt;">Findings:</span></h1>
<h2><span style="font-size:16pt;">Mobility Variables</span></h2>
<p><span style="font-size:11pt;">The following section discusses descriptive findings and makes comparisons among routing and network length variables, respectively.&nbsp;</span></p>
<h3><span style="color:#434343;font-size:13.999999999999998pt;">Routing Variables</span></h3>

Figure 3: Choropleth maps of distance to closest BART station, bus stop, and BayWheels port

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/56532c81-b61e-4ae4-89f5-a1f9b94880b7" width="32%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/6a76a75f-d16f-4dad-b82f-f62b428e6326" width="32%" /> 
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/b0df216c-d862-4636-976f-f8700cc29029" width="32%" />
</p>

Our routing variables color code each Census Block based on how close the centroid is to the nearest mobility asset. Each map is color coded based on 10 quantiles with the dark purple representing the nearest distance to the transportation facility and yellow representing the farthest distance. Distribution and quantiles can be further explored in histograms, as seen in Figure 4. 

Figure 4: Histograms of distance to closest BART station, bus stop, and BayWheels port

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/ea869989-b72d-494b-9582-20250b330d1a" width="32%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/677f0d0f-b572-4486-bda0-8cd065d38827" width="32%" /> 
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/cb35e027-7f08-4068-815d-fc498835b3ec" width="32%" />
</p>

Choropleth maps and histograms indicate widely varying spatial distributions for our three routing variables. The BART map shows clear purple clusters around each of the nine stations in our area of analysis, with gradually brighter colors radiating out from the station. The Bay Wheels map is more dispersed and shows how there are distinct areas and corridors that have good bike share coverage, particularly in the Northern and Western zones of the region. The Southern and Eastern zones have little to no bike share coverage. The bus stop choropleth map is significantly more heterogeneous and the distribution shows much closer distances when compared to BART and BayWheels, which implies that the bus network has the widest coverage of the three modes. Proximate blocks have different colors while the rail and bikeshare maps have clear nodes and the colors are in a consistent gradient. This pattern is due to the smaller differences in quantiles due to the steeper distribution of close stops. 



<h3><span style="color:#434343;font-size:13.999999999999998pt;">Network Length Variables</span></h3>

Figure 5: Choropleth maps of walk- and bike-to-drive network lengths

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/7ba0e6e4-7132-4397-b530-7d442ddbf30b" width="49%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/d1c709e4-c12f-41dd-8345-fbf0631d6dcb" width="49%" /> 
</p>

These choropleth maps color code each Census Block based on the given ratio, with the dark purple representing the lowest ratio of walk/bike network to drive network and yellow highest ratio. Distribution and quantiles can be further explored in histograms, as seen in Figure 6. 

Figure 6: Histograms of walk-to-drive and bike-to-drive network ratios

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/98238a6d-fb69-4aa2-a738-6b79264a3d3c" width="49%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/f47e6e9e-f143-4fa8-a682-1876fc483533" width="49%" /> 
</p>

As can be seen in the figures above, the distribution of network ratio variables was quite different from the distribution of routing variables. These variables both had a large number of Blocks with very low ratios, and a steep drop off. The distribution for the walk-to-drive ratio is consistently greater than 1, which suggests that nearly all Blocks have as much if not more walkway than road. Overall the bike-to-drive ratios are much smaller, and over half of the Blocks have a bike-to-drive ratio of 0. This demonstrates the much more sparse network of bike lanes and paths. As can be seen in the histogram, the walk-to-drive ratio in particular had some very large outliers. These outliers are likely parks and special areas like the UC Berkeley campus and Lake Merritt that have little to no driving roads and large networks of walking or biking paths. Therefore we set an artificial maximum value to be 10 and adjusted any values larger than this, which ended up being only 0.2% of the Blocks. 

<h2><span style="font-size:16pt;">Mobility Index</span></h2>


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Routing Only</span></h3>

Figure 7: Histogram of Scores - Weighted for Distance to Transit
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/6fcd5e7f-fbaa-4bc8-b794-3d01e1d651e6" width="75%" />
</p>

The above histogram shows the distribution of transit-oriented scores. Scores are within the range of 1 to 15. The vast majority of scores are 11 or higher which suggests that the East Bay has strong access to transportation. Few Blocks have scores below 9.

Figure 8: Map of Scores - Weighted for Distance to Transit
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/d7d00a8d-11d7-4170-8cd3-aec7f87653aa" width="75%" />
</p>

The above map shows the Census Blocks colored by their transit index score. Purple indicates the least access to BART, bus, and bike share and yellow indicates the most. There are clear clusters of yellow around the BART stations which are accentuated by the high number of bus stops that serve as connections to the rail network. One disconnect between the histogram and the map is that there is a higher number of small Census Blocks that are clustered around the main transportation corridors. The high percentage of total blocks having high scores in the histogram is not reflected in the map that shows a comparatively lower percentage of land area having high scores. 


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Network Ratios Only</span></h3>

Figure 9: Histogram of Scores - Weighted for Walk/Bike Network Density
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/a9e27dba-4ecf-4b10-8cad-b6aceb699ea1" width="75%" />
</p>

The above histogram shows the score distribution for the network density oriented index. It shows the spread of Census Blocks with scores weighted to highlight the walking and biking network density relative to the driving network density. In contrast to the transit oriented score, the chart indicates that most Blocks have a score between 2 and 5 out of 10. A high-level takeaway from this difference is that the East Bay has a stronger transit network than it does active transportation infrastructure.

Figure 10: Map of Scores - Weighted for Walk/Bike Network Density
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/e8c267b0-ea6d-4a36-bae3-96a4430d517a" width="75%" />
</p>

The above map shows the spectrum of walk and bike network density scores represented spatially. Each Block is color coded by score with purple representing low and yellow representing high scores. Compared to the transit figures, the histogram and map are more in sync. In terms of spatial distribution, there are visible segments of the East Bay that have far denser active transportation networks. Berkeley and North Oakland have a noticeable higher index score than the rest of the region. There is also a highly scored corridor starting at Lake Merritt and following the waterfront toward Fruitvale and Oakland Coliseum. The Eastern segment of the area has visibly lower scores, which is likely due to the steep hills and lower density land use where sidewalks and bike lanes are less common.


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Combination Score</span></h3>

Figure 11: Histogram of Scores - Combination
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/fa647b78-8b5b-4a8d-921b-dd4893666416" width="75%" />
</p>

The above histogram shows the spread of scores for our most important index, which incorporates all five transportation metrics. The histogram is more closely aligned with the transit oriented index because these metrics overall are given more weight than the walk and bike networks due to their greater ability to move people throughout the region. Our index indicates that East Bay has a fairly high level of mobility access. The majority of Blocks have scores between 10 and 13 out of a maximum of 15. There are outlier Blocks, but most have combined mobility levels in the top third of possible scores.

Figure 11: Map of Scores - Combination
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/b4dd5636-f2fe-4df0-b2ba-902eb5ce662b" width="75%" />
</p>

Our combined score map indicates that the highest score Blocks are in line with the BART corridor in the area of analysis that starts in North Berkeley and moves South to Oakland Coliseum. The pockets of yellow are concentrated around these stations. This is because BART was given the highest weight in the index due to its ability to travel to more of the Bay Area than bus or train. The pink and lighter purple blocks indicate greater distance to rail but strong access to bus and bike share, and may also indicate a high presence of bike lanes. The yellow pockets are also bolstered by good bus connectivity to the BART network. The darker purple areas are larger, less populated blocks and includes parks and other areas that transit cannot serve. This explains why the overall percentage of high-score land area is lower than the percent of Blocks with high scores seen in the histogram. At the same time, there are small, dense blocks in the Southeast part of Oakland that are darker shades of purple suggesting high population but low mobility access.


<h2><span style="font-size:16pt;">Comparison to LIHTC Properties</span></h2>


Figure 12: Map of Scores - LIHTC Properties over Combination-scored Census Blocks
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/3e8a574b-cd16-4990-ba38-d33192119519" width="75%" />
</p>


The above map shows the location of LIHTC properties overlaid on the combination score map. The affordable housing points appear to be situated primarily in blocks with high mobility scores. There is a large cluster of properties near Downtown Oakland and Lake Merritt and a high number of properties dispersed throughout Berkeley and North Oakland where mobility scores are high. There are a few properties in South Oakland that are in low score Blocks, but the map suggests that LIHTC properties are overall located in areas with good mobility access, which is an indication that their residents are well situated to access jobs, services, amenities, and other points of interest throughout the Bay Area. 


<p><span style="font-size:11pt;">Average scores for the 161 Blocks with LIHTC properties in them:</span></p>
<ul>
  <li>Berkeley: 13.59</li>
  <li>Oakland: 12.52</li>
  <li>Emeryville: 12.95</li>
</ul> 
             
Score distribution for all Blocks in the East Bay:
<ul>
 <li>count: 6890</li>
 <li>mean: 11.900425</li>
  <li>min: 1.232193</li>
  <li>25%: 10.907036</li>
  <li>50%: 12.399435</li>
  <li>max: 15.023467</li>
</ul> 

We found that across all three cities, the average mobility score for blocks with affordable housing was higher than the average score for the full East Bay. Oakland and Emeryville properties both had average scores between the 50th and 75th percentile of scores for all blocks in the East Bay. Berkeley properties had an average score above the 75th percentile for the region. Overall, Berkeley properties were the most accessible, but by a small margin.



<h1><span style="font-size:20pt;">Discussion:</span></h1>

<h2><span style="font-size:16pt;">Implications of Findings</span></h2>

Our analysis demonstrated that overall access to mobility across the East Bay is strong, but there is clear spatial inequality across different sections. While the average scores for Census Blocks is high, our maps show that there are areas with small, dense Census Blocks (that suggests high population density) where access to transportation options are limited. This is most visible in Southern and Eastern Oakland, which is likely the reason why Oakland’s average score for LIHTC locations is lower than in Berkeley and Emeryville. 

We also noticed that Berkeley has very high mobility scores in nearly the entire city and especially near the UC campus. The campus is a commute hub for its 28,000 employees and 45,000 students who live throughout the East Bay. Numerous AC Transit lines converge at the campus and pass through neighboring parts of the city. The corridor from Downtown Oakland to the campus is notably accessible due to the BART route, higher concentration of bike share ports, and AC transit route patterns mentioned above. The distribution of distance to bus stops was the most concentrated of the three modes of transit. Our histogram suggests that most Blocks in the East Bay are within 500 meters of a bus stop. With a greater concentration of bus stops between Downtown Oakland and UC Berkeley compared to other parts of the East Bay, it makes sense that this is the most accessible section based on our mobility index. 

In evaluating the transportation network of the East Bay, our descriptive findings suggest that the network is well designed and has provided strong access on average. Closer analysis of our maps shows that there are certain areas that are lacking. Future transportation planning should take this into consideration and prioritize increasing access in the denser areas of East Oakland before adding improvements to the sections with already high access levels. This extends beyond transit service to active transportation networks. Our map showing only network density scores has a similar pattern to the transportation only map which indicates that East Oakland is also less walkable and bikeable than other parts of the East Bay. Laying the groundwork for accessible communities helps neighborhoods grow and thrive. 

For LIHTC property locations, there is a similar implication. While the average scores for property Blocks across all three cities were quite high, there are clusters of properties that are in relatively inaccessible areas. Our map of property points shows how East Oakland affordable housing locations are in meaningfully less accessible Blocks. The majority of properties being in high-access Blocks raises the average but obscures the issue of low scored blocks with LIHTC properties which are visible in the map. New affordable housing should be placed in Blocks with as high a score as possible, and steps should be taken to increase mobility in areas with dense populations but low access to transportation options.



<h2><span style="font-size:16pt;">Limitations and Next Steps</span></h2>

Limitations in our data and methods could inform potential next steps for this project. For instance, the spatial distributions of our network length variables were heavily skewed, and indicated that that a ratio might not provide the best metric of overall walkability or bikeability of an area. The Blocks with the highest ratios were those where there were a significant number of recreational bike trails but fewer roads, indicating that these may not be frequently used for commuting to work or elsewhere. In the future, it may be better to consider the cumulative walk or bike network length for a certain radius around the Block centroid, rather than considering the ratio. It would be interesting to see how closely spatial patterns correlate between cumulative network length and ratio as well, to see if this makes a considerable difference.

It may have also proved useful to remove outliers or set artificial maximums for more than just one variable. We did this only in the case of the walk-to-drive ratio, which was the most heavily skewed. Removing outliers would be less effective, however, as that would leave holes in our index. However, due to steep distributions for other variables such as the distance to the closest bus stop, some variables were more significant to differences in mobility index among Block Groups. Since there was much more variation in distance to BART, for example, this variable was automatically weighted heavier in a way that we were unable to quantify. Future work could include developing a standard procedure for identifying an artificial maximum cutoff value to avoid removing outliers and create more comparable distributions. 

For our metric of bus accessibility, it may have also been more impactful to calculate bus stop or bus route density for a given area rather than just distance to the closest bus stop. Not all bus stops have the same number of routes, have the same frequency, or cover the same service area. Just because a bus stop may be right outside one’s home does not mean that they will use that stop, as opposed to the one a few blocks away that takes them where they want to go. And even if a stop is close to a Block centroid, if there are no other stops nearby it may be less accessible than a Block with stops slightly further but much more dense. If this metric were to be included in the future, it should be considered at a larger scale than just the Block. The rationale for this is similar to why we measured network ratios at a larger scale. 

Additionally, there were many other variables we could have considered here that may factor into how multimodal a place is considered to be. While we considered factors of the transportation facilities, we did not factor in other land uses. Creating and factoring in variables related to housing density, points of interest, places of employment, and other built environment characteristics not directly related to the transportation network, could provide more nuance in our index. 

We also did not factor in people’s experiences of safety using these transportation facilities. Just because two blocks have a similar walk-to-drive ratio does not immediately indicate that they have equal levels of walkability when one area might have higher pedestrian collisions. Developing metrics for these concerns could better inform our understanding of the accessibility of a place. 

