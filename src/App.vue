<script>
const d3 = {
  ...require('d3'),
  ...require('d3-geo'),
  ...require('d3-tile'),
  ...require('d3-selection')
};

const MIN_ZOOM = 10;
const MAX_ZOOM = 27;

export default {
  props: {
    center: {
      type: Array,
      default: () => [2.1, 45],
    },
    initialZoom: {
      type: [Number, String],
      default: 14,
    },
  },
  computed: {
    projection () {
      return d3.geoMercator()
        .scale(+this.scale / (2 * Math.PI))
        .translate([this.translateX, this.translateY])
        .center(this.center)
      ;
    },
    tiles () {
      return d3.tile()
        .size([this.width, this.height])
        .scale(+this.scale)
        .translate(this.projection([0, 0]))()
      ;
    },
  },
  watch: {
    zoom (zoom, prevZoom) {
      const k = zoom - prevZoom > 0 ? 2 : .5;

      this.scale = 1 << zoom;
      this.translateY = this.height / 2 - k * (this.height / 2 - this.translateY);
      this.translateX = this.width / 2 - k * (this.width / 2 - this.translateX);
    },
  },
  methods: {
    onTouchStart (e) {
      this.touchStarted = true;

      this.touchLastX = e.clientX;
      this.touchLastY = e.clientY;
    },
    onTouchEnd () {
      this.touchStarted = false;
    },
    onTouchMove (e) {
      if (this.touchStarted) {
        this.translateX = this.translateX + e.clientX - this.touchLastX;
        this.translateY = this.translateY + e.clientY - this.touchLastY;

        this.touchLastX = e.clientX;
        this.touchLastY = e.clientY;
      }
    },
    zoomIn () {
      this.zoom = Math.min(this.zoom + 1, MAX_ZOOM);
    },
    zoomOut () {
      this.zoom = Math.max(this.zoom - 1, MIN_ZOOM);
    },
  },
  data () {
    return {
      width: 1424,
      height: 1000,
      translateX: 0,
      translateY: 0,

      touchStarted: false,
      touchLastX: 0,
      touchLastY: 0,

      zoom: +this.initialZoom,
      scale: 1 << +this.initialZoom,
    };
  },
  async mounted () {
    this.translateX = this.width / 2;
    this.translateY = this.height / 2;

    const svg = d3.select("svg"),
      width = +svg.attr("width"),
      height = +svg.attr("height");

    const data = await d3.json("https://raw.githubusercontent.com/holtzy/D3-graph-gallery/master/DATA/world.geojson");
    // Draw the map
    svg.append('g').selectAll("path")
      .data(data.features)
      .enter().append("path")
        .attr("fill", "#69b3a2")
        .attr("d", d3.geoPath()
            .projection(this.projection)
        )
        .style("stroke", "#fff");
  },
  render () {
    return (
      <div class="map">
        <div class="map__controls"> 
          <button
            class="map__button"
            disabled={this.zoom >= MAX_ZOOM}
            onClick={this.zoomIn}
          >+</button>
          <button
            class="map__button"
            disabled={this.zoom <= MIN_ZOOM}
            onClick={this.zoomOut}
          >-</button>
        </div>
        <svg
          viewBox={`0 0 ${this.width} ${this.height}`}
          onMousedown={this.onTouchStart}
          onMousemove={this.onTouchMove}
          onMouseup={this.onTouchEnd}
          onMouseleave={this.onTouchEnd}
        >
        </svg>
      </div>
    )
  }
};
</script>

<style lang="scss" scoped>
.map {
  width: 100%;
  height: 100%;
  position: relative;
  font-family: Arial, sans, sans-serif;

  &__tile {
    // reset pointer events on images to prevent image dragging in Firefox
    pointer-events: none;
  }

  &__controls {
    position: absolute;
    left: 16px;
    top: 16px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 56px;
  }

  &__button {
    border: 0;
    padding: 0;
    width: 24px;
    height: 24px;
    line-height: 24px;
    border-radius: 50%;
    font-size: 18px;
    background-color: #ffffff;
    color: #343434;
    box-shadow: 0 1px 4px rgba(0, 0, 0, .4);

    &:hover,
    &:focus {
      background-color: #eeeeee;
    }

    &:disabled {
      background-color: rgba(#eeeeee, .4);
    }
  }
}
</style>