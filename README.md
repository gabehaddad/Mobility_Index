# Mobility_Index
Categorization of Census blocks in the East Bay based on their mobility access


<h1><span style="font-size:20pt;">Introduction:</span></h1>
<p><span style="font-size:11pt;">The purpose of this project is to visualize access to transportation across the East Bay. Our goal was to identify which areas have the highest and lowest measures of access to non-auto mobility. Access to urban mobility is a key socioeconomic metric of neighborhoods in large cities. Our intent was to evaluate the spatial equity of the transportation network across our area of analysis.&nbsp;</span></p>
<p><span style="font-size:11pt;">To do so, we developed a series of indices that rank each Census Block in Berkeley, Oakland, and Emeryville based on their proximity to transportation facilities and the density of the transportation network. We first calculated access to nearby bus stops, BART stations, and Bay Wheels bike share docks along the walk network. Each of these modes serves a different need. BART is a regional rail network that can provide longer distance trips across the Bay Area region. These trips across the region can be in excess of 30 miles. AC Transit serves municipalities within Alameda County and has a far denser network of stations than BART. It is the primary public transit network for short to medium-distance trips within the East Bay zone we analyzed. Bay Wheels is a privately provided bike share service that is best suited for short distance trips as a substitute for walking. We then factored in the density of the pedestrian and bike infrastructure within the vicinity of each Block. By factoring in proximity to rail, bus, bike share, and active transportation infrastructure, our index paints a comprehensive picture of the mobility level of each Census Block in the region.</span></p>
<p><span style="font-size:11pt;">We also wanted to see how our own mobility ranking compares to government initiatives to promote spatial equity. Access to active transportation facilities, as well as short, medium, and long distance transit options is crucial in megaregions like the Bay Area. Land use patterns and high costs of driving in the region necessitate access to both local and regional transit, especially for low-income individuals and families. To evaluate how the transit and active transportation networks serve low-income communities, we mapped the locations of Low Income Housing Tax Credit (LIHTC) affordable housing properties. Overlaying these locations on top of our mobility index helps us understand the level of mobility access that affordable housing residents have. We selected the LIHTC program because it is by far the largest affordable housing program both nationally and in the Bay Area. LIHTC properties account for over 75% of overall affordable housing units in the East Bay, and have the most available data for spatial analysis. We plotted the location of these properties on top of our mobility index to paint a visual picture of where they are relative to high and low mobility areas of the region and calculated the average scores of the Census Blocks that each property sits on broken out by each of the three cities we analyzed.&nbsp;</span></p>
<p><span style="font-size:11pt;">Overall, we found that Berkeley is slightly more accessible than Oakland and Emeryville. We also found that LIHTC properties are, on average, located in high-mobility areas, which is an indicator that these residents have good access to transportation relative to the overall level of service in their cities. Areas of lower accessibility include East Oakland and the Oakland Hills.</span></p>


<h1><span style="font-size:20pt;">Methodology:</span></h1>
<p><span style="font-size:11pt;">We chose Census Blocks as our unit of spatial analysis because we wanted the most granular zones available without being too computationally heavy. Given the density of non-auto transportation facilities across the area of analysis, Blocks that are next to each other can have meaningfully different mobility access. In addition, as we only dealt with built environment characteristics, we did not have to worry about data not being available at this small of a spatial level. We considered using individual properties to maximize the precision of our index, but the number of calculations required was too high.&nbsp;</span></p>
<p><span style="font-size:11pt;">For our area of research interest, we chose to examine the East Bay, which we defined to be the cities of Oakland, Piedmont, Emeryville, and Berkeley. Within our study area, there were 6890 Census Blocks for which we did each of our calculations.&nbsp;</span></p>

<p><span style="font-size:11pt;">We calculated and included the following variables in our index:</span></p>
<ul>
    <li style="list-style-type:disc;font-size:11pt;">
        <p><span style="font-size:11pt;">Routing Variables:</span></p>
        <ul>
            <li style="list-style-type:circle;font-size:11pt;">
                <p><span style="font-size:11pt;">Distance to the closest BART station along the walking network</p>
            </li>
            <li style="list-style-type:circle;font-size:11pt;">
                <p><span style="font-size:11pt;">Distance to the closest bus stop along the walking network</p>
            </li>
            <li style="list-style-type:circle;font-size:11pt;">
                <p><span style="font-size:11pt;">Distance to the closest BayWheels &nbsp;bike share station</p>
            </li>
        </ul>
    </li>
    <li style="list-style-type:disc;font-size:11pt;">
        <p><span style="font-size:11pt;">Network Length Variables:</span></p>
        <ul>
            <li style="list-style-type:circle;font-size:11pt;">
                <p><span style="font-size:11pt;">Ratio of the length of the walk to road network</p>
            </li>
            <li style="list-style-type:circle;font-size:11pt;">
                <p><span style="font-size:11pt;">Ratio of the length of the bike to road network</p>
            </li>
        </ul>
    </li>
</ul>
<h2><span style="font-size:16pt;">Data Collection &amp; Wrangling</span></h2>
<p><span style="font-size:11pt;">The following table describes all datasets used and their sources. It can be assumed that all data used was the most recent data provided by these sources.&nbsp;</span></p>

<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/785d6d30-645b-4f82-87ab-0d265050b513" width="75%" />
</p>


Once we clipped the Census Blocks to the research area of interest, we removed any Blocks that contained only water and then found the centroid of each. All datasets were stored as GeoDataFrames with CRS set to World Geodetic System 1984 (EPSG 4326). 


Figure 1: Map of BART stations, AC Transit stops, and Bay Wheels ports
<p align="center">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/7107797e-e320-4c11-8a62-b81d5304b305" title="East Bay Mobility Assets">
</p>


<h3><span style="color:#434343;font-size:13.999999999999998pt;">Routing Variables</span></h3>
<p><span style="font-size:11pt;">To rank access to transportation, we calculated the distance from the centroid of each Block to the nearest BART station, bus station, and Bay Wheels port. We calculated the distance along the walk network, as we assumed (or hoped) that most people would be walking to these transportation facilities rather than driving. We created a walk network for the East Bay region using OSMnx, which used the OverPass API to access OpenStreetMap with a predefined query for walkable facilities. Though we used OSMnx to define our network, it was much more computationally heavy to use for routing calculations, and therefore we converted the OSMnx network to a Pandana type network to do routing with this package instead.&nbsp;</span></p>
<p><span style="font-size:11pt;">Because the Block centroids and transportation facility locations are typically not already located on the network, we used the Pandana get_node_id function to snap each origin (Block centroid) and destination (transportation facility) point to the network by adding this node ID information to our datasets. For each of our three routing variables, we then iterated through Census Blocks to find the distance from each origin to all possible destinations and selected the minimum distance among these. Even using Pandana this proved to be computationally complex, with over 15 million total routing calculations for the distance to bus stop variable alone. These minimum network distance variables were added to our Census Block dataset.&nbsp;</span></p>
<h3><span style="color:#434343;font-size:13.999999999999998pt;">Network Length Variables</span></h3>
<p><span style="font-size:11pt;">To score the walk and bike networks for a given Census Block, we calculated the ratio of walk and bike networks to the road network, respectively. We assume that higher ratios mean more options for active transportation routes relative to the driving network. For instance, a walk-to-drive ratio of 2.0 means that there are twice as many opportunities in an area to walk as opposed to drive.&nbsp;</span></p>

Figure 2: Maps of Walk, Bike, and Drive networks
<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/d399e349-06d0-4db4-90b2-f60ca7d7f10a" width="32%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/99b1ecf4-180c-48cd-b88b-6032079c5a1f" width="32%" /> 
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

Our routing variables color code each Census Block based on how close the centroid is to the nearest mobility asset. Each map is color coded based on 10 quantiles with the dark purple representing the nearest distance to the transportation facility and yellow representing the farthest distance. Distribution and quantiles can be further explored in histograms, as seen in Figure XX. 

Figure 4: Histograms of distance to closest BART station, bus stop, and BayWheels port

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/ea869989-b72d-494b-9582-20250b330d1a" width="32%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/677f0d0f-b572-4486-bda0-8cd065d38827" width="32%" /> 
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/cb35e027-7f08-4068-815d-fc498835b3ec" width="32%" />
</p>

Choropleth maps and histograms indicate widely varying spatial distributions for our three routing variables. The BART map shows clear purple clusters around each of the nine stations in our area of analysis, with gradually brighter colors radiating out from the station. The Bay Wheels map is more dispersed and shows how there are distinct areas and corridors that have good bike share coverage, particularly in the Northern and Western zones of the region. The Southern and Eastern zones have meaningfully less bike share coverage. The bus stop map is significantly more heterogeneous, which implies that the bus network has the widest coverage of the three modes. Proximate blocks have different colors while the rail and bikeshare maps have clear nodes and the colors are in a consistent gradient. This pattern is due to the higher number of stops. (Add more when we have histograms)

<p><span style="color:#434343;font-size:13.999999999999998pt;">Network Length Variables</span></p>

Figure 5: Choropleth maps of walk- and bike-to-drive network lengths

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/7ba0e6e4-7132-4397-b530-7d442ddbf30b" width="49%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/d1c709e4-c12f-41dd-8345-fbf0631d6dcb" width="49%" /> 
</p>

Figure 6: Histograms of walk-to-drive and bike-to-drive network ratios

<p float="left">
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/98238a6d-fb69-4aa2-a738-6b79264a3d3c" width="49%" />
  <img src="https://github.com/gabehaddad/Mobility_Index/assets/105304214/f47e6e9e-f143-4fa8-a682-1876fc483533" width="49%" /> 
</p>











