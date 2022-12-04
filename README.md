# Population-sensitive travel distance between regions

## Overview

This notebook aims to calculate travel distances between different regions.
Since the regions are more extensive, we can't just take the center of the area as a starting/ending point: It is essential to know where people are traveling from.
Simplification: We assume that everyone in one region visits another equally often. That means we neglect the effect that border regions visit neighboring regions more frequently than other parts of an origin region.
Furthermore,  we always route via a particular road near Monte Ceneri since other data (Swisscom) is measured there.


Step by step:
1. Import regions and population density
2. Divide regions into squares with side lengths of 300m
3. Assign squares to regions
4. Calculate the population in squares:
Absolute population and relative population per region
5. select dense squares: Select only the most populated squares per region. Select squares to include 90% of the population.
6. Calculate distance region A --> region B :
   1. Calculate all distances from dense squares in A to dense squares in B
   2. Average distances weighted by the product: relative_population_in_square_A * relative_population_in_square_B.
7. export as csv

## Installation Quick-Start

### Requirements

- container virtualization [Docker](https://docs.docker.com/) to run [Valhalla](https://github.com/valhalla/valhalla)
- development environment [Java Development Kit](https://docs.oracle.com/en/java/javase/18/install/overview-jdk-installation.html#GUID-8677A77F-231A-40F7-98B9-1FD0B48C346A) to run [OpenTripPlanner](https://github.com/opentripplanner/OpenTripPlanner/tree/dev-1.x)
- programming language [Python 3.11](https://www.python.org/downloads/) to run the main script as an [Jupyter notebook](https://jupyter.org/)
- Install python packages with ```pip: -r requirements.txt```

### Valhalla: car routing

```bash
docker run -dt --name valhalla_gis-ops -p 8002:8002 -v $PWD/custom_files:/custom_files -e tile_urls=https://download.geofabrik.de/europe/switzerland-latest.osm.pbf gisops/valhalla:latest
```

### OpenTripPlanner: ÖV routing

for the 2019 schedule:

```bash
cd src/OpenTripPlanner
java -jar otp-1.5.0-shaded.jar --build graphs/ch2019_tiny_TI/ --inMemory --port 2019 --securePort 8019
```

for the 2022 schedule:

```bash
cd src/OpenTripPlanner
java -jar otp-1.5.0-shaded.jar --build graphs/ch2022_tiny_TI/ --inMemory --port 2022 --securePort 8022
```

Output details:

- population density is aggregated into a 300m square grid
  - 80% of the whole population is represented by a selection of 3% of all squares which is 13% of the populated squares

![square selection](_output/Zellenauswahl.png)

- ÖV
  - The routing is based on the [GTFS](https://opentransportdata.swiss/en/cookbook/gtfs/) data from 2019 and 2022
  - The routing is multimodal for public transport (ÖV) combined with pedestrian
  - It is routed directly between the regions without via Monte Ceneri
  - The distance includes the complete route
  - Time includes only travel time not walking time
- Car
  - For verification purposes, the accessibility from Chiasso was calculated. The route was not explicitly routed via Monte Ceneri. The verification plots are in the folder ```tests/```
