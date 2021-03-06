## Description
`This is a reverse-proxy service for NextBus.`

## Required Software
`Docker`<br/>
`docker-compose`<br/>
*       curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
        chmod +x /usr/local/bin/docker-compose
`Python 2.7`

## How to start
`docker-compose up`

## Scaling
`docker-compose up --scale web=<number>`

## Design

The API calls have been segregated into 3 high level groups. <br/>
*       config (/api/v1/config)
        The config endpoint supports 3 commands: agencyList, routeList, routeConfig
            agencyList:  /api/v1/config?command=agencyList
            routeList:   /api/v1/config?command=routeList&a=<agency_tag>
            routeConfig: /api/v1/config?command=routeConfig&a=<agency_tag>, or
                         /api/v1/config?command=routeConfig&a=<agency_tag>&r=<route_tag>
                          
        
*       prediction (/api/v1/prediction)
        The prediction endpoint supports 3 commands: predictions, predictionsForMultiStops and schedule
        predictions: /api/v1/prediction?command=predictions&a=<agency_tag>&stopId=<stop id>, or
                     /api/v1/prediction?command=predictions?command=predictions&a=<agency_tag>&stopId=<stop id>&routeTag=<route tag>
        predictionsForMultiStops: /api/v1/prediction?command=predictionsForMultiStops&a=<agency_tag>&stops=<stop_tag>&stops=<stop_tag> (User can specify multiple stops)
        schedule: /api/v1/prediction?command=schedule&a=<agency_tag>&r=<route_tag>
        
*       message (/api/v1/message)
        The message endpoint supports 2 commands: messages and vehicleLocations
        messages: /api/config/message?command=messages&a=<agency_tag>, or
                  /api/config/message?command=messages&a=<agency_tag>&r=<route_a>&r=<route_2> (User can specify multiple routes)
        vehicleLocations: /api/config/message?command=vehicleLocations&a=<agency_tag>&r=<route_tag>&t=<time in epoch>
                  
                  
`The reverse-proxy service is hosted on docker containers and is made scalable with the use of 
docker-compose and a round-robin scheduling algorithm provided by the docker image dockercloud/haproxy.
The load balancer is linked to the service in the docker-compose.yml file. There is a database container
that is also associated with the service. It stores the stats for all requests made to NextBus`

## How to terminate

` docker-compose down -v`<br/><br/>
` The -v option is to make sure that all volumes are freed once the service is stopped. 
This helps avoid errors is starting up the mongodb container due to dangling volumes`

## Error Handling
`Checks are in place to ensure that only those request are sent to the NextBus server which contain all 
required parameters. If a request is missing some of the required parameters, the service 
returns an error message and does not make any request to the NextBus servers`

## Limitations

`In order to prevent some users from being able to download so much data that 
it would interfere with other users we have imposed restrictions on data usage. These limitations could change at any time. They currently are:`
*   Maximum characters per requester for all commands (IP address): 2MB/20sec
*   Maximum routes per "routeConfig" command: 100
*   Maximum stops per route for the "predictionsForMultiStops" command: 150
*   Maximum number of predictions per stop for prediction commands: 5
*   Maximum time-span for "vehicleLocations" command: 5min

`For slow endpoints, the api/v1/stats endpoint returns only the latest occurence
of the slow endpoint`


## Assumptions

*   Threshold for slow endpoints is assumed to be .5 seconds. This means that any request that 
takes more than .5 seconds is classified as a slow endpoint.
*   The user is aware of the NextBus API contracts listed here
*       http://www.nextbus.com/xmlFeedDocs/NextBusXMLFeed.pdf

