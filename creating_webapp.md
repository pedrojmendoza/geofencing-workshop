The geo-fencing management web application is based on [AWS Amplify](https://aws.amazon.com/amplify/) and specifically on React.

To avoid facing issues with the local depedency management, we recommend you to create a new [Cloud9](https://aws.amazon.com/cloud9/) environment as IDE. We used a t2.micro instance running Amazon Linux.

In order to create it, please follow the instructions detailed in the [Amplify React Tutorial](https://docs.amplify.aws/start/getting-started/installation/q/integration/react).

1. As part of the [Prerequisites](https://docs.amplify.aws/start/getting-started/installation/q/integration/react) you will be initializing Amplify and that will create a new AWS profile on your Cloud9 environment and these credentials will be used.

2. Once you are done with the *Prerequisites* step, feel free to [Set up fullstack project](https://docs.amplify.aws/start/getting-started/setup/q/integration/react) using the provided instructions.

3. Once you are done with the *Set up fullstack project* step and working on the [Connect the API and database to the app](https://docs.amplify.aws/start/getting-started/data-model/q/integration/react) section, you will be asked about the expiration of your API key, you might want to use a value larger than 7 days for its expiration (we used 1 year expiration instead). 

Before the initial *amplify push* to deploy the backend, please replace the contents of the **amplify/backend/api/<YOUR_API_NAME>/schema.graphql** file with the following definition:

```
type Geofence @model {
  id: ID!
  name: String!
  geometry: String
}
```

Feel free to skip (or adapt) the *Test your API* section (as it refers to the TODO entity testing rather than the Geofence).

Next, install the dependecies require to include [Leaflet](https://leafletjs.com/) and [React Leaflet](https://react-leaflet.js.org/) by running the following:

```
npm install react-leaflet-draw react-leaflet leaflet-draw leaflet
```

When working on the *Connect frontend to API* section, instead of the instructions on the tutorial, modify the **src/App.js** file and replace its contents with the below code:

```
/* src/App.js */
import React, { useEffect, useState } from 'react'

import { API, graphqlOperation } from 'aws-amplify'
import { createGeofence } from './graphql/mutations'
import { listGeofences } from './graphql/queries'
import { Map, Popup, TileLayer, FeatureGroup, Polygon } from "react-leaflet";
import { EditControl } from "react-leaflet-draw"
import "./App.css";

const App = () => {
  const [formState, setFormState] = useState()
  const [geofences, setGeofences] = useState([])
  const [activeGeofence, setActiveGeofence] = useState(null);

  // data operations
  useEffect(() => {
    fetchGeofences()
  }, [])

  function setInput(key, value) {
    setFormState({ ...formState, [key]: value })
  }

  async function fetchGeofences() {
    try {
      const geofenceData = await API.graphql(graphqlOperation(listGeofences))
      const geofences = geofenceData.data.listGeofences.items
      setGeofences(geofences)
    } catch (err) { console.log('error fetching geofences') }
  }

  return (
    <>
      <Map center={[45.4, -75.7]} zoom={11}>
        <TileLayer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          attribution='&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
        />

        <FeatureGroup>
          <EditControl
              position="topright"
              onCreated={e => {
                console.log(e);
                //console.log(JSON.stringify(e.layer.toGeoJSON().geometry.coordinates));
                // Reverse order of lat and lon
                let latlons = [];
                for (let lonlat of e.layer.toGeoJSON().geometry.coordinates[0]) {
                  let latlon = [];
                  latlon.push(lonlat[1], lonlat[0]);
                  latlons.push(latlon);
                }
                const enteredName = prompt('Please enter the region name')
                const geofence = {name: enteredName, geometry: JSON.stringify(latlons)}
                API.graphql(graphqlOperation(createGeofence, {input: geofence}));
                setGeofences([...geofences, geofence])          
              }}
              edit={{ remove: false, edit: false }}
              draw={{
                  marker: false,
                  circlemarker: false,
                  circle: false,
                  rectangle: false,
                  polygon: true,
                  polyline: false
              }}
          />

          {geofences.map(geofence => (
            <Polygon
              key={geofence.id}
              id={geofence.id}
              positions={JSON.parse(geofence.geometry)}
              onClick={() => {
                setActiveGeofence(geofence);
              }}
            />
          ))}

          {activeGeofence && (
            <Popup
              position={[
                JSON.parse(activeGeofence.geometry)[0][1],
                JSON.parse(activeGeofence.geometry)[0][0],
              ]}
              onClose={() => {
                setActiveGeofence(null);
              }}
            >
              <div>
                <h2>{activeGeofence.name}</h2>
              </div>
            </Popup>
          )}
        </FeatureGroup>
      </Map>
      <div>
        <input
          id="geometry"
          size="200"
          onChange={event => setInput('geometry', event.target.value)}
          value={activeGeofence ? activeGeofence.geometry : ""} 
          placeholder="Geometry"
          type="hidden"
        />
      </div>
    </>
  )
}

export default App
```

Additionally, you should modify the **public/index.html** file and add the following section to import the styles required by Leaflet. You should paste these just below the other *<link>* elements in the file.

```
    <link
      rel="stylesheet"
      href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
      integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
      crossorigin=""
    />    
    <link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/leaflet.draw/1.0.3/leaflet.draw.css"/>
```

And replace the contents of the **src/App.css** file with the following:

```
.leaflet-container {
  width: 100%;
  height: 90vh;
}

.sr-only {
  display: none;
}

.container { 
  width: 400;
  margin: '0 auto';
  display: 'flex';
  flex: 1;
  flexDirection: 'column';
  justifyContent: 'center';
  padding: 20 
}
```

You find face issues if you try to access http://localhost:8080 on Cloud9 (development server) so move on to the *Add authentication* section and then to *Deploy and host* section to directly see it working on a publicly accesible endpoint.

4. Once you are done with the above customizations, you are ready to [Add authentication](https://docs.amplify.aws/start/getting-started/auth/q/integration/react) and finally [Deploy and host app
](https://docs.amplify.aws/start/getting-started/hosting/q/integration/react).

5. After you are done with deploying the web app, go to the URL for the published application, create a new user to be able to access the application and, once in the application, draw a new geofence covering the Ottawa area (close to what is shown in this [picture](./Ottawa.png)) and save it with the name *Ottawa*
