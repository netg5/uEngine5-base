<template>
  <div>
    <geometry-element
      selectable
      movable
      resizable
      connectable
      deletable
      :id.sync="activity.tracingTag"
      :x.sync="activity.elementView.x"
      :y.sync="activity.elementView.y"
      :width.sync="activity.elementView.width"
      :height.sync="activity.elementView.height"
      :_style.sync="style"
      :parentId.sync="activity.elementView.parent"
      :label.sync="activity.name.text"
      v-on:dblclick="showProperty"
      v-on:selectShape="closeComponentChanger(); selectedActivity();"
      v-on:deSelectShape="closeComponentChanger(); deSelectedActivity();"
      v-on:removeShape="closeComponentChanger"
      v-on:redrawShape="closeComponentChanger"
      v-on:addedToGroup="onAddedToGroup"
    >
      <geometry-circle
        :center="[50,50]"
        :radius="50"
      >
      </geometry-circle>
            <geometry-circle
              :center="[50,50]"
              :radius="40"
              :_style="{
                                       'stroke-width': 1
                                     }"
              >
            </geometry-circle>

      <sub-elements>
        <bpmn-state-animation :status="status" :type="type"></bpmn-state-animation>

        <geometry-polygon
               :vertices="[[50,22],[25,66],[73,66]]"
                :_style="{
                         'stroke-width': 0.8,
                       }"
             ></geometry-polygon>


      </sub-elements>
      <bpmn-sub-controller :type="type"></bpmn-sub-controller>
    </geometry-element>

    <bpmn-property-panel
      :drawer.sync="drawer"
      :item.sync="activity"
    >
      <template slot="properties-contents">
        <md-input-container>
          <label>Event Name</label>
          <md-input type="text"
                    v-model="activity.name.text"></md-input>
        </md-input-container>
        <md-input-container>
          <label>retryDelay</label>
          <md-input type="number"
                    v-model.number="activity.retryDelay"></md-input>
        </md-input-container>
        <md-input-container>
          <label>Attached Activity ID</label>
          <md-input type="text"
                    v-model="activity.attachedToRef"></md-input>
        </md-input-container>

      </template>
      <template slot="additional-tabs">

      </template>
    </bpmn-property-panel>
  </div>
</template>

<script>
  import IBpmn from '../../IBpmn'
  export default {
    mixins: [IBpmn],
    name: 'bpmn-signal-intermediate-catch-event',
    props: {},
    computed: {
      defaultStyle(){
        return {
          'label-position': 'bottom',
          'label-width': 120,
          'stroke-width': 1.5,

        }
      },
      type(){
        return 'IntermediateEvent'
      },
      className(){
        return 'org.uengine.kernel.bpmn.SignalIntermediateCatchEvent'
      },
      createNew(newTracingTag, x, y, width, height){
        return {
          _type: this.className(),
          name: {
            text: ''
          },
          tracingTag: newTracingTag,
          selected: false,
          elementView: {
            '_type': 'org.uengine.kernel.view.DefaultActivityView',
            'id': newTracingTag,
            'x': x,
            'y': y,
            'width': width,
            'height': height,
            'style': JSON.stringify({})
          }
        }
      }
    },
    data: function () {
      return {};
    },
    watch: {},
    mounted: function () {

    },
    methods: {}
  }
</script>


<style scoped lang="scss" rel="stylesheet/scss">

</style>

