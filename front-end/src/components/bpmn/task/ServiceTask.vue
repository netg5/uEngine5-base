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
      <geometry-rect
        :_style="{
          'fill-r': 1,
          'fill-cx': .1,
          'fill-cy': .1,
          'stroke-width': 1.2,
          fill: '#FFFFFF',
          'fill-opacity': 0,
          r: '10'
        }"
      >
      </geometry-rect>

      <sub-elements>
        <image-element
          v-bind:image="service_image"
          :sub-width="'20px'"
          :sub-height="'20px'"
          :sub-top="'5px'"
          :sub-left="'5px'"
        >
        </image-element>

        <bpmn-loop-type :loopType="loopType"></bpmn-loop-type>
        <bpmn-state-animation :status="status" :type="type"></bpmn-state-animation>
      </sub-elements>
      <bpmn-sub-controller :type="type"></bpmn-sub-controller>
    </geometry-element>

    <bpmn-property-panel
      :drawer.sync="drawer"
      :item.sync="activity"
    >
      <template slot="properties-contents">
        <md-input-container>
          <label>액티비티 명</label>
          <md-input type="text"
                    v-model="activity.name.text"></md-input>
          <!--<md-button @click="changeName">change name</md-button>-->
        </md-input-container>
        <md-input-container>
          <label>역할 (추후 레인에서 자동설정)</label>
          <!--<md-select v-model="activity.role.name">-->
            <!--<md-option v-for="role in definition.roles"-->
                       <!--:key="role.name"-->
                       <!--:value="role.name">-->
              <!--{{ role.name }}-->
            <!--</md-option>-->
          <!--</md-select>-->

          <md-input type="text" readonly
                    v-model="activity.role.name"></md-input>

        </md-input-container>
        <md-input-container>
          <label>호출 URI 패턴</label>
          <md-input type="text"
                    v-model="activity.uriTemplate"></md-input>
        </md-input-container>
        <md-input-container v-if="links">
          <label>호출 서비스 선택</label>
          <md-select v-model="activity.uriTemplate" @change="giveJSONHint">
            <md-option v-for="(link, key) in links" :value="link.href">{{key}}</md-option>
          </md-select>

        </md-input-container>
        <md-input-container>
          <label>호출 메서드</label>
          <md-select type="text"
                     v-model="activity.method">

            <md-option value="GET">GET</md-option>
            <md-option value="POST">POST</md-option>
            <md-option value="DELETE">DELETE</md-option>
            <md-option value="PUT">PUT</md-option>
            <md-option value="PATCH">PATCH</md-option>

          </md-select>
        </md-input-container>
        <md-input-container if="'GET,DELETE'.indexOf(activity.method) == -1">
          <label>입력 데이터 (JSON template)</label>
          <md-textarea v-model="activity.inputPayloadTemplate"></md-textarea>

        </md-input-container>
        <p>결과 데이터 매핑</p>
          <bpmn-parameter-contexts
            :parameter-contexts="activity.outputMapping"
            :definition="definition"
          ></bpmn-parameter-contexts>

          <md-checkbox
                    v-model.number="activity.skipIfNotFound">리소스 없을 경우 (404) 오류 처리 하지 않음</md-checkbox>

        <md-input-container>
          <label>retryDelay</label>
          <md-input type="number"
                    v-model.number="activity.retryDelay"></md-input>
        </md-input-container>
      </template>
      <template slot="additional-tabs">

      </template>
    </bpmn-property-panel>
  </div>
</template>

<script>
  import IBpmn from '../IBpmn'
  export default {
    mixins: [IBpmn],
    name: 'bpmn-service-task',
    props: {},
    created: function(){
      if(!this.activity.role)
          this.activity.role = {name:''};
    },
    computed: {
      defaultStyle(){
        return {}
      },
      type(){
        return 'Task'
      },
      className(){
        return 'org.uengine.kernel.bpmn.ServiceTask'
      },
      createNew(newTracingTag, x, y, width, height){
        return {
          _type: this.className(),
          name: {
            text: ''
          },
          role: {
            name: ''
          },
          outputMapping: [],
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
      return {
          links: null,
          service_image: location.pathname + ((location.pathname == '/' || location.pathname.lastIndexOf('/') > 0) ? '' : '/') + 'static/image/symbol/Service.png'
      };
    },
    watch: {
      drawer: function (opened) {
        if (opened) {
          this.activity.role.name =
            this.bpmnVue.getWhereRoleAmIByTracingTag(this.activity.tracingTag);


          var me = this;
          var serviceId = this.activity.role.name;

          this.$root.codi('eureka/apps/' + this.activity.role.name).get()
            .then(function (response) {

              var homepage = response.data.application.instance[0].homePageUrl; //dont' use
              homepage = serviceId.toLowerCase(); //use path by zuul rather

              me.$root.codi(homepage).get().then(function(response2){
                  if(response2.data._links){
                    me.links = response2.data._links;
                  }
              });

            });



        }
      },
    },
    mounted: function () {

    },
    methods: {

//        changeName: function(){
//          this.activity.name.text= this.activity.name.text + "aaa";
//        },

        giveJSONHint: function(){

          var url = function(href) {
            var l = document.createElement("a");
            l.href = href;
            l.path = l.pathname.substr(1);

            try {
              l.path = l.path.split("%7")[0];
            }catch(e){}

            return l;
          };


          var link = url(this.activity.uriTemplate);

          var me = this;
          this.$root.codi(link.path).get().then(function(response){
              if(response.data._links.profile){
                  var link = url(response.data._links.profile.href);

                  me.$root.codi(link.path).get().then(function(response){

                    console.log(response.data);

                      if(response.data.alps.descriptors[0].descriptors){
                        var jsonHint = {};

                        response.data.alps.descriptors[0].descriptors.forEach(function(property){
                           jsonHint[property.name] = '<%='+property.name+'%>';
                        });

                        me.activity.inputPayloadTemplate = JSON.stringify(jsonHint, null, 2);
                        var temp = me.activity;
                        me.activity = null;
                        me.activity = temp;
                      }


                  });

              }

          });

        }

    }
  }
</script>


<style scoped lang="scss" rel="stylesheet/scss">
</style>
