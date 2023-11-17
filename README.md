# Geographic Data Processing for Medical Services Using RAAM

This repository contains a Jupyter Notebook for processing geographic data related to medical services. The notebook is designed to calculate distances between various locations, fetch GEOIDs, and perform RAAM (Rational Agent Access Model) analysis. RAAM analysis evaluates the total cost of access in terms of weighted travel and congestion, which is crucial for enhancing the accessibility and efficiency of medical services in a given state.

## Sources

The notebook utilizes various data sources, including:

- OSRM API: For calculating distances between locations.
- ESRI: Geographic Data
- US Census Bureau: Provides geographic data and API use.
- PySAL's Access library: For performing RAAM analysis. RAAM analysis helps in understanding the balance of travel and congestion costs in medical service accessibility. *More information about PySAL and its Access module can be found here: https://pysal.org/access/generated/access.raam.raam.html#access.raam.raam*


## Data Limitations

The notebook calculates distances using the centroid of each county as a reference point. While this approach provides a practical estimation for large-scale analyses, it's important to note that this method is an approximation. The actual geographical distance between specific locations within counties may vary. This approximation method is chosen for its computational efficiency and simplicity in large datasets, but users should be aware of its limitations in terms of precision.

## Use
1. Install the required packages from `requirements.txt`:
   
    ```
        pip install -r requirements.txt
    ```
3. Load data (geographic and providers):
   
    ```
        df1 = getCounties(state)
        df2 = getLocations(state, type)
    ```

3. Use OSRM API to create a distance matrix *(duration can also be used)*:
   
    ```
        coords = np.vstack((src_coords, dest_coords))
        coordinates_str = ";".join([f"{x},{y}" for x, y in coords])
        osrm_endpoint = f"http://router.project-osrm.org/table/v1/driving/{coordinates_str}?sources={sources}&destinations={destinations}&annotations=distance"
        headers = requests.utils.default_headers()
        headers.update({'User-Agent': 'My User Agent 1.0'})
    
        attempts = 0
        while attempts < max_retries:
            try:
                response = requests.get(osrm_endpoint, headers=headers)
                response.raise_for_status()
                return json.loads(response.content)['distances']
            except requests.exceptions.RequestException as err:
                print(f"Attempt {attempts + 1} failed: {err}")
                attempts += 1
                time.sleep(delay)
   ```

5. Get the providers GEOID using Census Bureau API:
   
   ```
   
        base_url = "https://geocoding.geo.census.gov/geocoder/geographies/coordinates"
        params = {
            "x": longitude,
            "y": latitude,
            "benchmark": "Public_AR_Census2020",
            "vintage": "Census2010_Census2020",
            "format": "json",
            "key": API_KEY
        }
   
   ```
7. Create Supply/Demand/Times Tables:
   
    ```
        getProviders(provider, id_field, provider_type, state)
        getTimes(provider, id_field, provider_type, state)
        getPop(state, provider_type)
    ```
7. Run RAAM:
   ```
       def RAAMDis(A):
        A.raam(name="raamDis", tau=60)
        A.raam(name="raam30Dis", tau=30)
        A.access_df.sort_values(by=f"raamDis_{type}").dropna().head()
        A.score(name="raamDis_combo", col_dict={f'raamDis_{type}': 0.8})
        return A
   ```
8. Save Results to CSV:
   ``` 
       df = distanceA.norm_access_df
       df.to_csv(f"./{type}/RAAM/{state}_{type}_distance_results.csv") 
   ```
## Results
The results showcase the counties that have better and worse access relative to other counties within the analysis. 


### These are some images generated from RAAM results:

![ARLAMS](/ARLAMS.jpg)  
![OH](/OH.jpg)  


_Note: The images were generated using ArcGIS, and this code does not contain the steps for this._
