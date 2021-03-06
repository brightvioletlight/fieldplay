<template>
  <div class='settings' :class='{collapsed: settingsPanel.collapsed}'>
    <div class='block vector-field'>
      <div class='title'>Vector field <a class='reset-all' :class='{"syntax-visible": syntaxHelpVisible}' href='#' @click.prevent='syntaxHelpVisible = !syntaxHelpVisible'>syntax help</a></div>
      <syntax v-if='syntaxHelpVisible' @close='syntaxHelpVisible = false'></syntax>
      <pre>
<span class='comment'>// p.x and p.y are current coordinates
// v.x and v.y is a velocity at point p</span>
function velocity(<span class='type' >vec2</span> p) {
  <span class='type'>vec2</span> v = <span class='type'>vec2</span>(0., 0.);</pre>
      <textarea ref='codeInput' v-model='vectorField' type='text' rows='3' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></textarea>
<pre>  return v;
}</pre>
      <div class='error=container'>
        <pre v-if='error' class='error hl'>{{error}}</pre>
        <pre v-if='errorDetail' class='error detail'>{{errorDetail}}<span v-if='isFloatError'>
Did you forget to add a dot symbol? E.g. <span class='hl'>10</span> should be <span class='hl'>10.</span> and <span class='hl'>42</span> should be <span class='hl'>42.</span>
</span></pre>
      </div>
    </div>
    <form class='block' @submit.prevent='onSubmit'>
      <div class='title'>Settings<a class='reset-all' href='#' @click='reset'>reset all</a> </div>
      <div class='row'>
        <div class='col'>Particle color</div>
        <div class='col full'> 
          <select v-model='selectedColorMode' @change='changeColor'>
              <option value='1'>Uniform</option>
              <option value='2'>Velocity</option>
              <option value='3'>Angle</option>
	        </select>
        </div>
        <help-icon @show='selectedColorHelp = !selectedColorHelp' :class='{open: selectedColorHelp}'></help-icon>
      </div>
      <div class='row help' v-if='selectedColorHelp'>
        <div>
          <p>Defines background color for a vector field zone. Each particle entering into this zone wll be colored accordingly</p>
          <ul>
            <li><i>Uniform color</i> gives all particles the same color</li>
            <li><i>Velocity color</i>  makes particles "hotter" if they move faster, and "colder" if they move slower. Notable exception is when you have singularities in field. Then all colors are the same.</li>
            <li><i>Angle color</i> highlights zones based on velocity vector angle.</li>
          </ul>
          <p>Default value is "Uniform"</p>
        </div>
      </div>
      <div class='row' v-if='soundAvailable'>
        <div class='col'>SoundCloud track</div>
        <div class='col full'>
          <input type='text' v-model='soundCloudLink'>
          <a href='#' @click.prevent='loadSound'>load</a>
        </div>
      </div>
      <div class='row' v-if='soundAvailable'>
        <audio ref='player' controls='' autoplay='' preload autobuffer></audio>
      </div>
      <div class='row'>
        <div class='col'>Particles count </div>
        <div class='col full'><input type='number' :step='particleCountDelta' v-model='particlesCount' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='particleCountHelpVisible = !particleCountHelpVisible' :class='{open: particleCountHelpVisible}'></help-icon>
      </div>
      <div class='row help' v-if='particleCountHelpVisible'>
        <div>
          <p>How many particles should be visible inside bounding box? Higher values produce denser plots, smaller values are faster to compute.</p>
          <p>Recommended value is between <b>10,000</b> and <b>100,000</b></p>
        </div>
      </div>
      <div class='row'>
        <div class='col'>Fade out speed</div>
        <div class='col full'><input type='number' :step='fadeoutDelta'  v-model='fadeOutSpeed' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='fadeoutDeltaHelp = !fadeoutDeltaHelp' :class='{open: fadeoutDeltaHelp}'></help-icon>
      </div>
      <div class='row help' v-if='fadeoutDeltaHelp'>
        <div>
          <p>Before a particle is moved to the next position, we multiply its transparency by this number. This gives a fading out trace behind the particle</p>
          <ul>
            <li>Setting this value to <b>1</b> will keep particle trace forever.</li>
            <li> Setting this value to <b>0</b> will leave no trace at all</li>
          </ul>
          <p>Recommended value is <b>0.998</b></p>
        </div>
      </div>
      <div class='row'>
        <div class='col'>Particle reset probability</div>
        <div class='col full'><input type='number' :step='resetProbabilityDelta'  v-model='dropProbability' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        <help-icon @show='resetProbabilityHelp = !resetProbabilityHelp' :class='{open: resetProbabilityHelp}'></help-icon>
      </div>
      <div class='row help' v-if='resetProbabilityHelp'>
        <div>
          <p>This is a probability that a particle will reset its position to a random location inside bounding box. This prevents particles from flying out of the screen.</p>
          <ul>
            <li>Setting this value to <b>1</b> will reset all particles on every frame. This can be a good option to "reset" an empty screen.</li>
            <li>Setting this value to <b>0</b> will prevent particles from jumping to a random spot. This can be a good option to trace particles trajectory.</li>
          </ul>
          <p>Default value is <b>0.009</b></p>
        </div>
      </div>
      <div class='row'>
        <div class='col'>Integration timestep</div>
        <div class='col full'><input type='number' :step='integrationStepDelta' v-model='timeStep' @keyup.enter='onSubmit' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false" ></div>
        <help-icon @show='integrationStepHelp = !integrationStepHelp' :class='{open: integrationStepHelp}'></help-icon>
      </div>
      <div class='row help' v-if='integrationStepHelp'>
        <div>
          <p>This parameter defines how fast time flies for each particle (or, to be more accurate, this is the integration step of the classical Runge-Kutta method)</p>
          <ul>
            <li>Increasing this value makes particles fly faster at risk of missing proper curve's turns.</li>
            <li>Making this value smaller increases the accuracy of particle's trajectory, and makes them move slower.</li>
          </ul>
          <p>Default value is <b>0.01</b></p>
        </div>
      </div>
      <div class='bounding-box'>
        <div class='col title'>bounds</div>
        <div class='row'>
          <div class='col  center'><input type='number' v-model.lazy='minY' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
        <div class='row'>
          <div class='col min-x'><input type='number' v-model.lazy='minX' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
          <a class='col reset' href='#' @click.prevent='goToOrigin'>go to origin</a>
          <div class='col max-x'><input type='number' v-model.lazy='maxX' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
        <div class='row center'>
          <div class='col center'><input type='number' v-model.lazy='maxY' autocomplete="off" autocorrect="off" autocapitalize="off" spellcheck="false"></div>
        </div>
      </div>
    </form>
  </div>
</template>
<script>
import bus from '../lib/bus';
import appState from '../lib/appState';
import autosize from 'autosize';
import generateFunction from '../lib/generate-equation';
import SoundLoader from '../lib/sound/soundLoader';
import SoundCloudAudioSource from '../lib/sound/audioSource';
import config from '../lib/config';
import Syntax from './help/Syntax';
import HelpIcon from './help/Icon';

// Temporary disable this until API is finished.
const soundAvailable = config.isAudioEnabled;

export default {
  name: 'Settings',
  props: ['scene'],
  components: {
    Syntax,
    HelpIcon
  },
  mounted() {
    bus.on('scene-ready', this.onSceneReady, this);
    bus.on('generate-field', this.generateNewFunction, this);
    bus.on('bbox-change', this.updateBBox, this);
    bus.on('glsl-parser-result-changed', this.updateParserResults, this);
    autosize(this.$refs.codeInput);

    if (soundAvailable) this.soundLoader = new SoundLoader(this.$refs.player);
  },
  beforeDestroy() {
    bus.off('scene-ready', this.onSceneReady, this);
    bus.off('generate-field', this.generateNewFunction, this);
    bus.off('bbox-change', this.updateBBox, this);
    bus.off('glsl-parser-result-changed', this.updateParserResults, this);
    autosize.destroy(this.$refs.codeInput);
  },
  data() {
    return {
      error: '',
      errorDetail: '',
      soundCloudLink: 'https://soundcloud.com/mrfijiwiji/yours-truly',
      isFloatError: false,
      vectorField: '',
      settingsPanel: appState.settingsPanel,
      particlesCount: 0,
      fadeOutSpeed: 0,
      dropProbability: 0,
      timeStep: 0,
      selectedColorMode: 0,
      soundAvailable: soundAvailable,
      // TODO: Need something better for help management?
      selectedColorHelp: false,
      syntaxHelpVisible: false,
      particleCountHelpVisible: false,
      fadeoutDeltaHelp: false,
      resetProbabilityHelp: false,
      integrationStepHelp: false,
      minX: 0, minY: 0,
      maxX: 0, maxY: 0
    };
  },
  watch: {
    vectorField(newValue, oldValue) {
      console.log(newValue, oldValue);
      // TODO: this seem to be causing double initialization
      if (this.pendingSetCode) {
        clearTimeout(this.pendingSetCode);
      }

      this.pendingSetCode = setTimeout(() => {
        this.sendVectorField();
        this.pendingSetCode = 0;

        autosize.update(this.$refs.codeInput);
      }, 300);
    },
    particlesCount(newValue, oldValue) {
      this.scene.setParticlesCount(parseInt(newValue, 10));
    },
    timeStep(newValue, oldValue) {
      this.scene.setIntegrationTimeStep(newValue);
    },
    fadeOutSpeed(newValue, oldValue) {
      this.scene.setFadeOutSpeed(newValue);
    },
    dropProbability(newValue, oldValue) {
      this.scene.setDropProbability(newValue);
    },
    selectedColorMode(newValue) {
      this.scene.setColorMode(newValue);
    },
    minX(newValue) { this.moveBoundingBox('minX', newValue) },
    maxX(newValue) { this.moveBoundingBox('maxX', newValue) },
    minY(newValue) { this.moveBoundingBox('minY', newValue) },
    maxY(newValue) { this.moveBoundingBox('maxY', newValue) },
  },
  computed: {
    particleCountDelta() {
      return exponentialStep(this.particlesCount);
    },
    integrationStepDelta() {
      var timeStep = this.timeStep;
      return exponentialStep(timeStep);
    },
    resetProbabilityDelta() {
      return exponentialStep(this.dropProbability);
    },
    fadeoutDelta() {
      var fadeOutSpeed = Number.parseFloat(this.fadeOutSpeed);

      var exp = Math.round(Math.log10(1 % fadeOutSpeed)) ;
      var dt = Math.pow(10, exp);
      if (dt + fadeOutSpeed >= 1) {
        dt /= 10;
      }
      return dt;
    }
  },
  methods: {
    moveBoundingBox(key, value) {
      if (this.ignoreBbox) {
        return;
      } 
      this.scene.moveBoundingBox({[key]: value});
    },
    loadSound() {
      if (!this.soundLoader) return;
      this.soundLoader.loadStream(this.soundCloudLink).then(e => {
        if (!this.audioSource) {
          this.audioSource = new SoundCloudAudioSource(this.$refs.player); 
        }
        this.audioSource.playStream(this.soundLoader.streamUrl())
      });
      // TODO: Error handling
    },
    generateNewFunction() {
      this.vectorField = generateFunction();
    },
    reset() {
      // we reset the scene bounding box, and let the a.href = # do the rest.
      this.scene.resetBoundingBox();
    },
    goToOrigin() {
      this.scene.resetBoundingBox();
    },  
    onSubmit() {
      if (window.innerWidth < 600) {
        appState.settingsPanel.collapsed = true;
      }
    },

    changeColor(e) {
      this.selectedColorMode = e.target.value;
    },

    updateBackground(rgba) {
      this.scene.setBackgroundColor(rgba);
    },

    onSceneReady(scene) {
      this.vectorField = scene.getCurrentCode();
      this.particlesCount = scene.getParticlesCount();
      this.fadeOutSpeed = scene.getFadeOutSpeed();
      this.dropProbability = scene.getDropProbability();
      this.timeStep = scene.getIntegrationTimeStep();
      this.selectedColorMode = scene.getColorMode();
      this.updateParserResults(scene.getLastParserResult());
      this.updateBBox();
    },

    updateBBox() {
      this.ignoreBbox = true;
      var bbox = scene.getBoundingBox();
      this.minX = bbox.minX;
      this.maxX = bbox.maxX;

      // Y is weird in my implementation. I know..
      this.minY = bbox.minY;
      this.maxY = bbox.maxY;
      if (this.prevBboxReset) clearTimeout(this.prevBboxReset);

      this.prevBboxReset = setTimeout(() => {
        this.ignoreBbox = false
        this.prevBboxReset = 0
      }, 50);
    },

    sendVectorField() {
      let parserResult = this.scene.updateVectorField(this.vectorField);
      this.updateParserResults(parserResult);
    },

    updateParserResults(parserResult) {
      if (parserResult && parserResult.error) {
        this.error = parserResult.error;
        this.errorDetail = parserResult.errorDetail;
        this.isFloatError = parserResult.isFloatError;
      } else {
        this.error = '';
        this.errorDetail = '';
        this.isFloatError = false;
      }
    }
  }
}

function exponentialStep(value) {
  var dt = Math.pow(10, Math.floor(Math.log10(value)));
  if (value - dt === 0) {
    // This is odd case when you are increasing number, but otherwise it's a good adjustment.
    return dt/10;
  }
  return dt;
}

function toColorString({r, g, b, a}) {
  if (a === 1.0) {
    return `#${hex(r)}${hex(g)}${hex(b)}`;
  }
  return `rgba(${r}, ${g}, ${b}, ${a})`;
}

function hex(x) {
  let value = x.toString(16).toUpperCase();
  if (value.length === 1) value = '0' + value;
  return value;
}
</script>

<style lang='stylus'>
@import "./shared.styl";

help-background = rgb(7, 12, 23);

.settings {
  color: secondary-text;
  position: absolute;
  overflow-y: auto;
  left: 0;
  background: window-background;
  width: settings-width;
  border: 1px solid primary-border;
  border-left: none;
  border-top: none;
  padding: (7px + control-bar-height) 7px 7px 7px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  max-height: 100%;
}
.settings.collapsed {
  display: none;
}

.title {
  margin-bottom: 7px;
  color: primary-text;
  font-size: 18px;
}
.block {
  .col {
    align-items: center;
    display: flex;
  }
  .row {
    margin-top: 4px;
  }
  select {
    margin-left: 14px;
  }

  input[type='text'],
  input[type='number'] {
    background: transparent;
    color: primary-text;
    border: 1px solid transparent;
    padding: 7px;
    font-size: 16px;
    width: 100%;
    margin-left: 7px;
    &:focus {
      outline: none;
      border: 1px dashed;
      background: #13294f;
    }
  }
}

.help {
  margin: -7px;
  margin-bottom: 7px;
  padding: 7px 7px 14px 7px;
  background: help-background;


  .close {

  }
}
.title {
  a {
    float: right;
    font-size: 12px;
    font-style: italic;
    color: #267fcd;
    height: 30px;
    margin: -5px;
    padding: 7px;
  }

  a.syntax-visible {
    background: help-background;
    color: white;
    font-style: normal;
  }
}
form.block {
  margin-top: 12px;
  padding-top: 10px;

}
.vector-field {
  pre {
    margin: 0;
    color: #6789ab;
    .comment {
      color: #435970;
      font-style: italic;
    }
  }

  pre.error {
    color: rgba(250, 232, 55, 1);
    overflow-y: auto;
  }
  pre.error.detail {
    overflow: none;
    white-space: normal;
    .hl {
      background-color: #172A4D;
      color: red;
      font-weight: bold;
    }
  }

  textarea {
    max-height: 180px;
    background: transparent;
    color: white;
    font-family: monospace;
    margin-top: 14px;
    padding: 0;
    padding-left: 14px;
    width: settings-width - 14px;
    font-size: 18px;
    border: 1px solid transparent;
    &:focus {
      outline: none;
      border: 1px dashed;
      background: #13294f;
    }
  }
}

.row {
  display: flex;
  flex-direction: row;
}

.center {
  justify-content: center;
}

audio {
  width: 100%;
}

.col {
  flex: 1;
}
a {
  text-decoration: none;
}

a.action {
  color: white;
  font-size: 16px;
}

a.help-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 25px;
  margin-right: -7px;
  svg {
    fill: secondary-text;
  }
  &.open {
    background: help-background;
    svg {
      fill: primary-text;
    }
  }
}
.row.help {
  margin-top: 0;
}

.reset {
  text-decoration: none;
  color: white;
  display: flex;
  justify-content: center;
}

.bounding-box {
  position: relative;
  .title {
    position: absolute;
    bottom: 0;
    font-size: 12px;
    left: 0;
    color: #435970;
  }
  .reset {
    font-size: 16px;
  }

  input[type='number'] {
    width: 100px;
    margin: 0;
    font-size: 12px;
    text-align: center;
    color: secondary-text;
  }
  .max-x {
    justify-content: flex-end;
  }
}

@media (max-width: small-screen) {
  .settings {
    width: 100%;
    left: 0;
    .title {
      font-size: 14px;
      text-align: left;
    }
  }
  .vector-field {
    textarea {
      margin-top: 0;
      font-size: 16px;
      width: 100%;
    }
  }
}

</style>
