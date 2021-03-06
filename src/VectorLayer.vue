<template>
  <div class="vector-layer"><slot></slot></div>
</template>

<script>
import OSM from 'ol/source/osm'
import VectorLayer from 'ol/layer/vector'
import VectorSource from 'ol/source/vector'
import Feature from 'ol/format/feature'
import Style from 'ol/style/style'
import { parseFeatures, mapFeatureClasses } from './featureHelper'

export default {
  name: 'vector-layer',
  props: {
    source: {type: [String, Object], default: OSM},
    fitMapToThisLayer: {type: Boolean, default: false},
    opacity: {type: Number, default: 1},
    visible: {type: Boolean, default: true},
    zIndex: {type: Number, default: 1},
    projection: {type: String, default: 'EPSG:3857'},
    styleMap: {type: Object, default: _ => ({})},
    format: Function,
    extent: Array,
    // resolution boundaries in which the layer is visible
    maxResolution: Number,
    minResolution: Number
  },
  data () {
    return {
      layer: null
    }
  },
  mounted () {
    const defaultStyle = this.styleMap ? this.styleMap.default : Style.defaultFunction()
    const styleFunc = feature => {
      const mappedStyles = mapFeatureClasses(feature, this.styleMap)
      return mappedStyles.length ? mappedStyles : defaultStyle
    }
    const options = {
      opacity: this.opacity,
      visible: this.visible,
      zIndex: this.zIndex,
      extent: this.extent,
      style: styleFunc
    }

    if (this.format) {
      const features = parseFeatures(this.format, this.projection, this.source)
      const source = new VectorSource({ features })
      this.layer = new VectorLayer({ source, ...options })
    } else {
      this.layer = new VectorLayer({ source: this.source, ...options })
    }

    if (this.minResolution) {
      this.layer.setMinResolution(this.minResolution)
    }
    if (this.maxResolution) {
      this.layer.setMaxResolution(this.maxResolution)
    }

    this.$nextTick(_ => this.$parent.$emit('addLayer', {
      layer: this.layer,
      fitMapToLayer: this.fitMapToThisLayer
    }))
  }
}
</script>
