![Build](https://github.com/googlemaps/playablelocations-proxy/workflows/Build/badge.svg)
![Docker](https://github.com/googlemaps/playablelocations-proxy/workflows/Docker/badge.svg)
![Apache-2.0](https://img.shields.io/badge/license-Apache-blue)

# Playable Locations Proxy Server

A proxy implementation for the [Playable Locations API]. The implemenation allows converting latitude and longitude bounds to a single S2 cell id. Additionally, the proxy can be used extended to cache and route user requests through a central location.

This proxy makes use of the [Golang S2 package](https://github.com/golang/geo) for geometry conversion.

> **Note**: The proxy endpoint will overwrite the s2 cell id value in `area_filter.s2_cell_id` if `area_filter_lat_lng_bounds` is provided in requests to `/sampleplayablelocations`.

## Requirements
- [Go](https://golang.org/doc/install)
- [Docker](https://docs.docker.com/install/)(optional)

### Development

With Docker:

```
docker build . -t proxy
docker run proxy:latest
```

Alternatively, this can be built outside of Docker with normal Go tooling:
```
go mod download
go mod verify
go build -o proxy ./pkg
./proxy
```

The proxy server can be deployed with [Google Cloud Run](https://cloud.google.com/run/docs/deploying) using `gcr.io/geo-devrel-259418/playablelocations-proxy:latest`. Please use a digest when deploying an image, see [additional Docker images](https://console.cloud.google.com/gcr/images/geo-devrel-259418/GLOBAL/playablelocations-proxy?gcrImageListsize=30).

## Usage

### Sample Request via Curl

```sh
curl -X POST \
  localhost:8080/sampleplayablelocations \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/javascript' \
  -H 'x-goog-api-key: YOUR_API_KEY' \
  -d '{
  "criteria": [{
  	"game_object_type": 1234,
  	"fields_to_return": { "paths": ["name", "types", "snapped_point"] },
  	"filter": {
  		"max_location_count": 2,
  		"included_types": ["retail"]
  	}
  }],
  "area_filter_lat_lng_bounds": {
    "northeast": { "latitude": 37.761419193645686, "longitude": -122.41189956665039 },
    "southwest": { "latitude": 37.75714420786591, "longitude": -122.41790771484375 }
  }
}
'

```
### JSON Response
```js
{
    "locations_per_game_object_type": {
        "1234": {
            "locations": [
                {
                    "name": "curatedPlayableLocations/ChIJC2juwih-j4ARvLq4f6Oyeuc",
                    "LocationId": {
                        "PlaceId": "ChIJC2juwih-j4ARvLq4f6Oyeuc"
                    },
                    "types": [
                        "retail"
                    ],
                    "center_point": {
                        "latitude": 37.77033960000001,
                        "longitude": -122.4119288
                    },
                    "snapped_point": {
                        "latitude": 37.7703263,
                        "longitude": -122.41195379999999
                    }
                },
                {
                    "name": "curatedPlayableLocations/ChIJkyP7ByZ-j4ARmLwtGpDTgjk",
                    "LocationId": {
                        "PlaceId": "ChIJkyP7ByZ-j4ARmLwtGpDTgjk"
                    },
                    "types": [
                        "retail"
                    ],
                    "center_point": {
                        "latitude": 37.7686504,
                        "longitude": -122.41297919999998
                    },
                    "snapped_point": {
                        "latitude": 37.7686515,
                        "longitude": -122.41295099999999
                    }
                }
            ]
        }
    },
    "ttl": {
        "seconds": 86400
    }
}
```
[Using the Playable Locations API](https://developers.google.com/maps/tt/games/using_playable_locations) provides the full specification and explanation of fields.

[Playable Locations API]: https://developers.google.com/maps/tt/games/overview_locations
