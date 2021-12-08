<template>
  <v-app>
    <v-app-bar
      :clipped-left="$vuetify.breakpoint.lgAndUp"
      app
      color="brown darken-3"
      dark
    >
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title
        style="width: 400px"
        min-width="400px"
        class="ml-0 pl-4"
      >
        <span class="hidden-sm-and-down">JDCatメタデータのお試し検索サイト</span>&nbsp;&nbsp;

      </v-toolbar-title>
<v-col class="text-left d-flex flex-row align-center" cols="1" sm="1">
</v-col>
<v-col class="text-left d-flex flex-row align-center" cols="3" sm="3">
      <v-text-field
        flat
        solo-inverted
        hide-details
        prepend-inner-icon="mdi-magnify"
        label="検索語入力後、Enter/Returnを押してください"
        class="hidden-sm-and-down"
        v-model="schKey"
        @keydown.enter.native="schAll(schKey)"
        id="schKeyForm"        
      ></v-text-field>

</v-col>
<v-col class="text-left d-flex flex-row align-center" cols="2" sm="2">
{{hitcount}}
</v-col>
</v-app-bar>
      <v-main>

<v-row class="d-flex flex-row md-11">
<!--<v-col class="order1" cols="1">
</v-col>-->
<v-col class="order1" cols="3">


  <template>
  <v-treeview
  :items="facets"
  v-model="distcheck"
  :open="open"
  dense
  selectable
  open-on-click
  return-object
  @input="schAll(schKey,'dist',distcheck)"
  >
  <template v-slot:label="{ item }">
  <v-badge :content="item.count" color="orange" inline left>{{item.name}}</v-badge>
  </template>
  </v-treeview>
</template>
<!--
 <v-treeview :items="facets" item-key="id" item-text="name" open-all>
      <template v-slot:prepend="{ Distributors }">
        <v-checkbox v-model="distcheck" v-if="!Distributors.children" />
      </template>
    </v-treeview>
    -->
  <!--  <v-checkbox v-for="distributor in distributors" 
      v-model="distcheck"
      :key = distributor.id
      :label="distributor"
      :value="distributor"
      @change="schAll(schKey,'dist',distcheck)"
    ></v-checkbox>
    -->
</v-col>
<v-col class="order2" cols="9">
<template>
  <v-tabs
    fixed-tabs
    background-color="brown"
    dark
  >
    <v-tab>
      Metadata
    </v-tab>
    <v-tab>
      Map
    </v-tab>
    <v-tab>
      Timeline
    </v-tab>
<v-tab-item>
        <v-card flat>
  <v-data-table
    :headers="headers"
    :items="hssmetadata"
    :items-per-page="15"
    class="elevation-1"
    item-key="id"
  >
    <template v-slot:item.uri="{ item }">
      <a :href="item.uri" target="_blank">{{item.urin}}</a>
    </template>
    <template v-slot:item.description="{ item }">
      <span v-html="item.description"></span>
    </template>
  </v-data-table>
  </v-card>
  </v-tab-item>  
  <v-tab-item>
        <v-card flat>
          <template>
         <l-map
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      style="height: 700px"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer
        :url="url"
        :attribution="attribution"
      />
<template>
      <l-marker v-for="(marker,index) in markerPlaces"
      :key="marker.id+index"
      :lat-lng="marker.latlon">
    
        <l-popup>
          <div class="primary--text">{{marker.name}}</div>
          <div v-for="(title, index) in marker.titles"
          :key="'tit-'+index">
      <a :href="title.uri" target="_blank">{{title.title}}</a>
          </div>
        </l-popup>  
      </l-marker>
</template>
      <!--
        </l-popup>  
      </l-marker>

      <l-marker :lat-lng="withTooltip">
        
        <l-tooltip :options="{ permanent: true, interactive: true }">
          <div @click="innerClick">
            I am a tooltip
            <p v-show="showParagraph">
              Lorem ipsum dolor sit amet, consectetur adipiscing elit. Quisque
              sed pretium nisl, ut sagittis sapien. Sed vel sollicitudin nisi.
              Donec finibus semper metus id malesuada.
            </p>
          </div>
        </l-tooltip>
        
      </l-marker>
      -->
  
    </l-map>
      </template>
     </v-card>
  </v-tab-item>
    <v-tab-item>
        <v-card flat>
  <timeline ref="timeline" :items="tlitems" :groups="tlgroups" :options="tloptions"> </timeline>
     <v-btn
  color="brown lighten-1 white--text"
  elevation="10"
  @click="clusterSwith"
>{{clustermsg}}</v-btn> 
    <v-alert
      text
      prominent
      type="error"
      icon="mdi-cloud-alert"
    >
項目のまとめの解除&nbsp;&nbsp;は、ヒット数が多い時に実行するとブラウザが固まりますのでご注意ください。
200件未満を目安にしていただくと安全かと思います。
</v-alert>

     </v-card>
  </v-tab-item>
  </v-tabs>
</template> 
  </v-col>
  </v-row>
<!--<div>aa {{ distcheck }}</div>-->
<!--<div @click="openJdcat">check_all_data</div>-->
&nbsp;&nbsp;<a href="https://jdcat.jsps.go.jp/">JDCatサイト</a>

</v-main>
  </v-app>
</template>

<script>
import jdcatData from './components/all_data3.json';
import menus from './components/menus.json';
import geoData from './components/geodata.json';
import "leaflet/dist/leaflet.css"
import { latLng } from "leaflet";
import { LMap, LTileLayer, LMarker, LPopup } from "vue2-leaflet";
//import { Timeline } from "vis-timeline";
import { Timeline } from 'vue-visjs'
//Vue.component('timeline', Timeline)
//import { LMap, LTileLayer, LMarker, LPopup, LTooltip } from "vue2-leaflet";


export default {
  name: 'App',
    components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    Timeline
    //LTooltip
  },

  data: () => ({
    tlgroups: [
      {
        id: 0,
        content: '簡易年表'
      }
    ],
    tlitems: [
      {
        id: 0,
        group: 0,
        start: '2011-01-01',
        end: '2013-01-01',
        content: 'Item 1'
      }
    ],
    tloptions: {
      height:'700px',
      cluster: {
      titleTemplate: "{count} 件のデータがまとめられています。",
      showStipes: true
      }
    },
    clustermsg:'項目のまとめの解除',
    markerPlaces:[],
    zoom: 3,
    center: latLng(35,135),
    url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
    attribution:
      '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
    withPopup: latLng(47.41322, -2.219482),
    withTooltip: latLng(47.41422, -1.250482),
    currentZoom: 11.5,
    currentCenter: latLng(35, 135),
    showParagraph: false,
    mapOptions: {
      zoomSnap: 0.5
    },
    showMap: true,
    open:['Root'],
    hitcount:'',
    distcheck:[],
    facets:[],
    latlon:[],
    jdcatAllData:jdcatData,
    drawer:false,
    menus:menus,
    schKey:'',
    headers: [
       {
          text: 'Title',
          align: 'start',
          value: 'title',
        },
       // { text: 'Title', value: 'title' },
        { text: 'Description', value: 'description' },
        { text: 'Detail', value: 'uri' },
        { text: 'Distributor', value: 'distributor' },
      ],
    hssmetadata:[{
      'title':'',
      'distributor':'',
      'description':'',
      'uri':'',
      'urin':''
      }
    ],
    hssmetadataEn:[{
      'title':'',
      'distributor':'',
      'description':'',
      'uri':'',
      'urin':''}
    ],    
  }),
  methods:{
    openJdcat: function(){
      console.log(jdcatData);
    },
    coLists: function(e,e2){
      let listCon = e.concat(e2);
      let result = listCon.filter((value, index, self) => self.indexOf(value) === index && self.lastIndexOf(value) !== index);
      return result.length;
    },
    sortDict: function(e){
      let array = Object.keys(e).map((k)=>({ key: k, value: e[k] }));
      array.sort((a, b) => b.value - a.value);
      e = Object.assign({}, ...array.map((item) => ({
            [item.key]: item.value,
      })));
      return e;
    },
    schAll: function(e,md,vals) {
      let _this = this;
      let geo_dict = {}
      geoData.jdcatgeo.forEach(function(e){
        geo_dict[e.name] = e
      })
      let hssmetadata = [];
      //const languages = ['ja','en','ja-Latn'];
      console.log(e);
      let re = new RegExp(e)
      let distributors = {};
      let markerPlaces = [];
      let topics = {};
      let geos = {};
      let geo_titles = {};
      let geo_title = {};
      let tl_title = {};
      let tl_titles = [];
      //let licenses = {};
      //let terms = {};
      jdcatData.jdcat.forEach(function(e2){
        let hflg = 0;
        let hssDict = {};
        //let hssDictEn = {};
        //console.log(e2);
        for (let field in e2){
          //console.log(e2[field])
          let e2field = e2[field];
          if(typeof e2field == "object"){
            if ("ja" in e2field){
              if(typeof e2field["ja"] == "string"){
                if(e2field["ja"].match(re)){
                  hflg = 1;
                }//if(e2field["ja"].match(re)){
              }//if(typeof e2field["ja"] == "string"){
            }//if ("ja" in e2field){
          }//if(typeof e2field == "object"){
        }//for (let field in e2){
        if(md == "dist"){
          if(hflg == 1){
          let vals2 = {}
          vals2["dist"] = [];
          vals2["topic"] = [];
          vals2["place"] = [];
          if(!e2["Geo"]){
            e2["Geo"] = {}
            e2["Geo"]["ja"] = [];
          }
          vals.forEach(function(e){
            if(e.id.match(/^dist-/)){
              vals2["dist"].push(e.name);
            }
            else if (e.id.match(/^topic-/)){
              vals2["topic"].push(e.name);
            }
            else if (e.id.match(/^place-/)){
              vals2["place"].push(e.name);
            }            
          })
          if(vals2["dist"].length == 0){//Dist
            if(vals2["topic"].length == 0){
              if(_this.coLists(e2["Geo"]["ja"], vals2["place"]) == 0){
                hflg = 0;
              }              
            }
            else if(vals2["place"].length == 0){
              if(_this.coLists(e2["Topic"]["ja"], vals2["topic"]) == 0){
                hflg = 0;
              }
            }
            else{
              if(_this.coLists(e2["Topic"]["ja"], vals2["topic"]) == 0 || _this.coLists(e2["Geo"]["ja"], vals2["place"]) == 0){
                hflg = 0;
              }
            }
          }
          else if(vals2["topic"].length == 0){//Topic
            if(vals2["dist"].length == 0){
              if(_this.coLists(e2["Geo"]["ja"], vals2["place"]) == 0){
                hflg = 0;
              }
            }
            else if(vals2["place"].length == 0){
              if(!vals2["dist"].includes(e2["Distributor"]["ja"])){
                hflg = 0;
              }
            }
            else{
              if(!vals2["dist"].includes(e2["Distributor"]["ja"]) || _this.coLists(e2["Geo"]["ja"], vals2["place"]) == 0){
                hflg = 0;
              }
            }
          }
          else if(vals2["place"].length == 0){//Place
            if(vals2["dist"].length == 0){
              if(_this.coLists(e2["Topic"]["ja"], vals2["topic"]) == 0){
                hflg = 0;
              } 
            }
            else if(vals2["topic"].length == 0){
              if(!vals2["dist"].includes(e2["Distributor"]["ja"])){            
                hflg = 0;
              }
            }
            else{
              if(_this.coLists(e2["Topic"]["ja"], vals2["topic"]) == 0 || !vals2["dist"].includes(e2["Distributor"]["ja"])){
                hflg = 0;
              }
            }
          }//else if(vals2["place"].length == 0){         
          else{
            if(!vals2["dist"].includes(e2["Distributor"]["ja"]) || _this.coLists(e2["Topic"]["ja"], vals2["topic"]) == 0 || _this.coLists(e2["Geo"]["ja"], vals2["place"]) == 0){
              //console.log(e2["Topic"]["ja"]);
              hflg = 0;
            }
          }
          }
        }// if(md == "dist"){
        if(hflg == 1){
          hssDict["title"] = e2["Title"]["ja"]
          hssDict["distributor"] = e2["Distributor"]["ja"]
          if("ja" in e2["Description"]){
            hssDict["description"] = e2["Description"]["ja"].replace(/\\n/g, '<br>')
          }
          else{
            let desc_list = [];
            for (let k3 in e2["Description"]){
              desc_list.push(k3+':'+e2["Description"][k3]["ja"])
            }
            hssDict["description"] = desc_list.join(', ');
          }
          hssDict["uri"] = 'https://jdcat.jsps.go.jp/records/'+e2["id"]
          hssDict['urin'] = 'JDCatサイトへ';          
          hssmetadata.push(hssDict);
          let e_distributor = hssDict['distributor'];
          if (!distributors[e_distributor]){
            distributors[e_distributor] = 0;
            console.log(e_distributor);
          }
          distributors[e_distributor] += 1
          e2["Topic"]["ja"].forEach(function(e6){
            if(!topics[e6]){
              topics[e6] = 0;
            }
            topics[e6] += 1;
          })
          if("Geo" in e2){
            geo_title = {};
            geo_title['uri'] = 'https://jdcat.jsps.go.jp/records/'+e2['id'];
            geo_title['title'] = e2['Title']["ja"];
            e2["Geo"]["ja"].forEach(function(e6){
              if(!geos[e6]){
                geos[e6] = 0;
                geo_titles[e6] = []
              }
              geo_titles[e6].push(geo_title);
              geos[e6] += 1;
            })
          }
          if("TimeP" in e2){
            tl_title = {};
            tl_title['uri'] = 'https://jdcat.jsps.go.jp/records/'+e2['id'];
            let tl_titlename = e2['Title']["ja"];
            tl_title['title'] = tl_titlename;
            e2["TimeP"].forEach(function(e6){
              if(!tl_titles[tl_titlename]){
                tl_titles[tl_titlename] = []
              }
              tl_title['date'] = e6;
              //console.log(e6);
              tl_titles.push(tl_title);
            })
          }                       
        }//if(hflg == 1){
      })
      this.hitcount = hssmetadata.length+' 件ヒット';
      this.hssmetadata = hssmetadata;
      if(md != 'dist'){
        /////////////////distここから//////////        
        let child_nodes = []
        let idno = 2;
        distributors = this.sortDict(distributors);
        for (let kd in distributors){
          let child_node = {}
          child_node["name"] = kd;
          child_node["count"] = distributors[kd];
          child_node["id"] = 'dist-'+idno;
          idno += 1;
          child_nodes.push(child_node);
        }
        let dist_dict = {}
        dist_dict["id"] = 'dist-1'
        dist_dict["name"] = "Distributors"
        dist_dict["children"] = child_nodes;
        dist_dict["count"] = child_nodes.length;
        this.facets = [];
        this.facets.push(dist_dict);
        /////////////////distここまで//////////
        /////////////////topicここから//////////        
        child_nodes = []
        idno = 2;
        topics = this.sortDict(topics);
        for (let kd in topics){
          let child_node = {}
          child_node["name"] = kd;
          child_node["count"] = topics[kd];
          child_node["id"] = 'topic-'+idno;
          idno += 1;
          child_nodes.push(child_node);
        }
        dist_dict = {}
        dist_dict["id"] = 'topic-1'
        dist_dict["name"] = "Topics"
        dist_dict["children"] = child_nodes;
        dist_dict["count"] = child_nodes.length;
        this.facets.push(dist_dict);
        /////////////////topicここまで//////////
        /////////////////geoここから//////////        
        child_nodes = []
        idno = 2;
        geos = this.sortDict(geos);
        let lat = ''
        let lon = ''
        let marker = {}
        for (let kd in geos){
          let child_node = {}
          child_node["name"] = kd;
          child_node["count"] = geos[kd];
          child_node["id"] = 'place-'+idno;
          idno += 1;
          child_nodes.push(child_node);
          lat = geo_dict[kd]['lat']
          lon = geo_dict[kd]['lon']
          //console.log(geo_titles[kd]);
          marker['name'] = kd;
          marker['titles'] = geo_titles[kd];
          marker['id'] = 'mk-'+idno;
        
          marker['latlon'] = latLng(lat,lon)
          markerPlaces.push(marker);
          //console.log(marker);          
          marker = {}
        }
        dist_dict = {}
        dist_dict["id"] = 'place-1'
        dist_dict["name"] = "Places"
        dist_dict["children"] = child_nodes;
        dist_dict["count"] = child_nodes.length;
        /////////////////geoここまで//////////   
        /////////////////timepここから//////////   
        //title, date, uri     
        let tl_nodes = [];
        let tl_node = {};
        let tl_dates = [];
        idno = 0;
        tl_titles.forEach(function(e){
          tl_dates = [];
          tl_node = {};
          tl_node['id'] = idno;
          tl_node['group'] = 0;
          tl_dates = e.date.split(':');
          if(tl_dates[0].length == 4){tl_dates[0] += '-01-01'}
          if(tl_dates[0].length == 7){tl_dates[0] += '-01'}          
          tl_node['start'] = tl_dates[0];
          if(tl_dates.length > 1){
            if(tl_dates[1].length == 4){tl_dates[1] += '-01-01'}
            if(tl_dates[1].length == 7){tl_dates[1] += '-01'}
            tl_node['end'] = tl_dates[1];          
          }

          tl_node['content'] = '<a target="_blank" href="'+e.uri+'">'+e.title+'</a>';
          
          console.log(tl_node);
          tl_nodes.push(tl_node);
          idno += 1;
        });
        this.tlitems = tl_nodes;
        /////////////////timepここまで//////////            
        this.facets.push(dist_dict);
        this.markerPlaces = markerPlaces;
      }
    },
    zoomUpdate(zoom) {
      this.currentZoom = zoom;
    },
    centerUpdate(center) {
      this.currentCenter = center;
    },
    showLongText() {
      this.showParagraph = !this.showParagraph;
    },
    clusterSwith(){
      if(this.tloptions.cluster.showStipes == true){
        this.tloptions.cluster = false;
        this.clustermsg = '近い項目をまとめる';
      }
      else{
        this.clustermsg = '項目のまとめの解除';
        this.tloptions.cluster = {titleTemplate: "{count} 件のデータがまとめられています。", showStipes: true};
      }
    }//clusterSwith(){
  },
  mounted: function(){
    this.schAll();
  }

};
</script>
<style>

    .vis-item .vis-item-overflow {
      overflow: visible;
    }

</style>