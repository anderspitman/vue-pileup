<template>
  <v-container>
    <v-card>
      <v-layout column>
        <v-layout row justify-space-between>
          <v-flex xs3>
            {{heading}}
          </v-flex>
          <v-layout xs3 justify-center>
            <v-btn fab small @click='zoomOut'>
              <v-icon>zoom_out</v-icon>
            </v-btn>
            <v-btn fab small @click='zoomIn'>
              <v-icon>zoom_in</v-icon>
            </v-btn>
          </v-layout>
          <v-flex xs3>
            <v-btn @click='launchFullIGV'>Open IGV in a Tab</v-btn>
          </v-flex>
        </v-layout>
        <v-flex xs12>
          <div id='igv-content'></div>
        </v-flex>
      </v-layout>
    </v-card>
  </v-container>
</template>

<script>

import igv from '../../lib/igv.esm'


export default {
  name: 'pileup',
  props: {
    heading: String,
    referenceURL: String,
    locus: String,
    visible: {
      type: Boolean,
      default: true,
    },
    tracks: Array,
    showLabels: {
      type: Boolean,
      default: false,
    },
  },
  data () {
    return {
      browser: null,
    }
  },
  mounted: function () {
    if (this.visible) {
      this.init();
    }
  },
  methods: {
    init: function() {
      const igvDiv = this.$el.querySelector('#igv-content');
   
      const options = {
        showControls: false,
        showIdeogram: true,
        showTrackLabels: this.showLabels,
        //showCenterGuide: true,
        minimumBases: 20,
        reference: {
         fastaURL: this.referenceURL,
        },
        locus: this.locus,
        tracks: [],
        //showCursorTrackingGuide: true,
        //showSequence: false,
        //showNavigation: false,
        //showRuler: false,
        //flanking: 1000,
        //locus: 'chr8:128748750-128749000',
      }

      for (const track of this.tracks) {

        //if (track.variantURL) {
        //  // insert variant track as first track
        //  options.tracks.push({
        //    name: track.name + " Variants", 
        //    type: "variant",
        //    format: "vcf",
        //    url: track.variantURL,
        //    //squishedCallHeight: 1,
        //    //expandedCallHeight: 4,
        //    //displayMode: "squished",
        //    visibilityWindow: 1000
        //  });
        //}
        
        options.tracks.push({
          height: 120,
          coverageTrackHeight: 30,
          alignmentRowHeight: 1,
          name: track.name, 
          type: 'alignment',
          url: track.alignmentURL,
          indexURL: track.alignmentIndexURL,
        });
      }

      igv.createBrowser(igvDiv, options).then((browser) => {
        this.browser = browser;
      })
    },
    zoomOut: function() {
      this.browser.zoomOut();
    },
    zoomIn: function() {
      this.browser.zoomIn();
    },
    launchFullIGV: function() {
      launchIGV(this.referenceURL, this.locus, this.tracks);
    },
  },
  watch: {
    visible: function() {
      if (!this.browser) {
        this.init();
      }
      else if (!this.visible) {
        igv.removeBrowser(this.browser);
        this.browser = null;
      }
      //igv.visibilityChange();
    },
  }
}

function launchIGV(referenceURL, locus, tracks) {

  const igvTracks = tracks.map((track) => ({
    type: 'alignment',
    //format: 'bam',
    url: track.alignmentURL,
    indexURL: track.alignmentIndexURL,
    name: track.name,
  }))

  const igvConfig = {
    showIdeogram: true,
    reference: {
      fastaURL: referenceURL,
    },
    locus,
    tracks: igvTracks,
  }

  if (tracks[0].variantURL) {
    igvConfig.tracks.unshift({
      name: 'Variants',
      type: 'variant',
      format: 'vcf',
      url: tracks[0].variantURL,
    })
  }

  const url = 'https://s3.amazonaws.com/static.iobio.io/dev/igv.iobio.io/index.html?config=' + JSON.stringify(igvConfig)
  window.open(url, '_blank')
}

</script>

<style>

.igv-right-hand-gutter {
  right: initial;
  left: -10px;
}

.igv-content-header {
  //display: none;
}

</style>
