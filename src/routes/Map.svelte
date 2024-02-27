<!-- Map.svelte -->
<script>
  import { onMount } from 'svelte';
  import 'ol/ol.css'; // Import OpenLayers CSS
  import Map from 'ol/Map';
  import View from 'ol/View';
  import TileLayer from 'ol/layer/Tile';
  import OSM from 'ol/source/OSM';
  import { Draw } from 'ol/interaction';
  import { Vector as VectorSource } from 'ol/source';
  import { Vector as VectorLayer } from 'ol/layer';
  import { Style, Fill, Stroke, Text } from 'ol/style';

  let map;
  let draw;
  let type = 'Polygon';

  onMount(() => {
      const raster = new TileLayer({
          source: new OSM(),
      });

      const source = new VectorSource({ wrapX: false });

      const vector = new VectorLayer({
          source: source,
      });

      map = new Map({
          layers: [raster, vector],
          target: 'map',
          view: new View({
              center: [-11687603.768526185, 4828527.173843127],
              zoom: 12,
          }),
      });

      addInteraction();
  });

  function addInteraction() {
      if (type !== 'None') {
          draw = new Draw({
              source: map.getLayers().getArray()[1].getSource(),
              type: type,
          });

          draw.on('drawend', function(event) {
              const feature = event.feature;
              const name = prompt('Enter name for the polygon:');
              const description = prompt('Enter description for the polygon:');
              // Attach name and description properties to the drawn feature
              feature.setProperties({
                  'name': name,
                  'description': description
              });

              // Create a fill style for the polygon shape
              const fillStyle = new Fill({
                  color: 'rgba( 221, 49, 12 , 0.2)'
              });

              // Create a stroke style for the polygon shape
              const strokeStyle = new Stroke({
                  color: 'black',
                  width: 2
              });

              // Apply the fill and stroke styles to the feature
              feature.setStyle(new Style({
                  fill: fillStyle,
                  stroke: strokeStyle
              }));

              // Create a text style for labeling
              const textStyle = new Text({
                  font: '14px Calibri,sans-serif',
                  fill: new Fill({
                      color: '#000'
                  }),
                  stroke: new Stroke({
                      color: '#fff',
                      width: 2
                  }),
                  text: name, // Use the name as the label text
                  offsetX: 0, // Adjust the offset if needed
                  offsetY: 10, // Adjust the offset if needed
              });

              // Apply the text style to the feature
              feature.getStyle().setText(textStyle);
          });

          map.addInteraction(draw);
      }
  }

  function handleTypeChange(event) {
      type = event.target.value;
      map.removeInteraction(draw);
      addInteraction();
  }

  function handleUndo() {
      draw.removeLastPoint();
  }
</script>

<div id="map" style="width: 100%; height: 100%;"></div>
