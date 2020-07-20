<template>
  <div></div>
</template>

<script>
import { loadModules } from "esri-loader";

export default {
  name: "web-map",

  data() {
    return {
      map: null,
      view: null,
      search: null,
    };
  },

  mounted: function () {
    loadModules(["esri/Map", "esri/views/MapView", "esri/widgets/Search"], {
      css: true,
    }).then(([ArcGISMap, MapView, Search]) => {
      // Map Init
      this.map = new ArcGISMap({
        basemap: "topo-vector",
      });

      this.view = new MapView({
        container: this.$el,
        map: this.map,
        center: [-118, 34],
        zoom: 8,
      });

      this.search = new Search({
        view: this.view,
      });

      // Modules Init
      this.view.ui.add(this.search, "top-right");

      // Listeners
      this.view.on("click", this.mapClickHandler);
    });
  },

  methods: {
    mapClickHandler(evt) {
      this.search.clear();
      this.view.popup.clear();
      if (this.search.activeSource) {
        let geocoder = this.search.activeSource.locator; // World geocode service
        let params = {
          location: evt.mapPoint,
        };
        geocoder.locationToAddress(params).then(
          (response) => {
            // Show the address found
            let address = response.address;
            this.showPopup(address, evt.mapPoint);
          },
          (err) => {
            // Show no address found
            console.error(err);
            this.showPopup("No address found.", evt.mapPoint);
          }
        );
      }
    },
    showPopup(address, pt) {
      this.view.popup.open({
        title:
          +Math.round(pt.longitude * 100000) / 100000 +
          "," +
          Math.round(pt.latitude * 100000) / 100000,
        content: address,
        location: pt,
      });
    },
  },

  beforeDestroy() {
    if (this.view) {
      // destroy the map view
      this.view.container = null;
    }
  },
};
</script>

<style scoped>
div {
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
}
</style>
