<template>
  <div class="hello">
    <div class="header">
      <div class="card shadow buttonContainer">
        <b-form-file v-model="csvFile">
    
        </b-form-file>
        <b-button v-on:click="getList" class="button">
          Get Product List
        </b-button>
    
      </div>
    </div>
    <div class="card shadow tableContainer">
      <b-input v-model="filterString" class="filterString mt-2 mb-2" card shadow placeholder="Search by name,email or role" >

      </b-input>
      
      <b-table :items="result1" striped hover :filter="filterString" id="table" :current-page="pageNo"
    :per-page="pagesize" @filtered="searchChangeHandler"  :fields="fields">

      <template #cell(batch)="data">

        <multiselect

        v-model="objectDetails[data.item.name].batchValue"
        :searchable="false"
        :close-on-select="true"
        :show-labels="false"
        placeholder="Pick a value"
        :allow-empty="false"
        :options="data.item.batch"
        v-on:input="changeRow(data.item.name)"
        >
      </multiselect>

      </template>

      </b-table>
      <div class="paginationContainer">
          <b-button class="buttonDeleteAll" variant="danger"> Delete All Selected</b-button>
          <b-pagination class="pagination" v-model="pageNo" :total-rows="resultLength" :per-page="pagesize"
          aria-controls="table" v-on:change="delselectHeader" ></b-pagination >
 

      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue'
const XLSX=require("xlsx")
      
// Import Bootstrap and BootstrapVue CSS files (order is important)
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

// Make BootstrapVue available throughout your project
Vue.use(BootstrapVue)

import { BootstrapVueIcons } from 'bootstrap-vue'
import 'bootstrap-vue/dist/bootstrap-vue-icons.min.css'

import Multiselect from 'vue-multiselect'

// register globally
// Vue.component('multiselect', Multiselect)
// import { json } from 'body-parser'

Vue.use(BootstrapVueIcons)

// Optionally install the BootstrapVue icon components plugin
Vue.use(IconsPlugin)
// import axios from "axios"
// import { filter } from "vue/types/umd"
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
  ,  
  components: {
    Multiselect
  },
  data(){
    return(
      {sampleArray:[1,2,3,4],
        result:undefined,
        filterString:"",
        pageNo:1,
        pagesize:10,
        resultLength:0,
        statusCheckAllRows:false,
        statusCheckRows:{},
        result1:[],
        csvFile:[],
        objectDetails:{},
        fields:["name","batch","stock","deal","free","mrp","rate","exp","supplier","company"],

        
      }
    )
  },
  methods:{
    async getList(){
      var selectedFile=this.csvFile
       var fileReader = new FileReader();
        fileReader.readAsBinaryString(selectedFile);
        fileReader.onload = (event)=>{
         let data = event.target.result;
         let workbook = XLSX.read(data,{type:"binary"});

         workbook.SheetNames.forEach(sheet => {
              let rowObject = XLSX.utils.sheet_to_row_object_array(workbook.Sheets[sheet]);
  
               console.log(rowObject)
               this.result=rowObject
               
         });
        }

  console.log("calling modify array function")
  setTimeout(()=>{this.modifyArray()},2000)
  },
  batchArray(prodName){
    return(this.objectDetails[prodName]["batches"])
  },
  getNameValue(nameValue){
    this.NameValue=nameValue
  },
  compareDates(date1,date2){
    if(date1>=date2){
      return(date1)

    }else{
      return(date2)
    }
  },
  changeRow(prodName){
    // console.log(this.value)
    const batch=this.objectDetails[prodName].batchValue
  const index=this.objectDetails[prodName]["sI"]
    if(batch==="All"){

      this.result1[index].stock=this.objectDetails[prodName]["All"].stock
    this.result1[index].deal=this.objectDetails[prodName]["All"].deal
    this.result1[index].rate=this.objectDetails[prodName]["All"].rate
    this.result1[index].mrp=this.objectDetails[prodName]["All"].mrp
    this.result1[index].free=this.objectDetails[prodName]["All"].free
    this.result1[index].exp=this.objectDetails[prodName]["All"].exp
    }
    else{
      var tempArray=this.objectDetails[prodName][batch]
      var resObject={        
          name:prodName,
          batch:this.objectDetails[prodName]["batches"],
          stock:tempArray[0].stock,
          deal:tempArray[0].deal,
          free:tempArray[0].free,
          mrp:tempArray[0].mrp,
          rate:tempArray[0].rate,
          exp:tempArray[0].exp,
          supplier:tempArray[0].supplier,
          company:tempArray[0].company

      }
   var temp1={}
    for (let i=1;i<tempArray.length;i++){
      temp1=tempArray[i]
      resObject.stock+=temp1.stock
      if(resObject.deal>temp1.deal){
        resObject.deal=temp1.deal
      }
      if(resObject.free>temp1.free){
        resObject.free=temp1.free
      }
      if(resObject.rate<temp1.rate){
        resObject.rate=temp1.rate
      }
      if(resObject.mrp<temp1.mrp){
        resObject.mrp=temp1.mrp
      }
      if(resObject.exp>temp1.exp){
        resObject.exp=temp1.exp
      }
    }
    // console.log
    this.result1[index].stock=resObject.stock
    this.result1[index].deal=resObject.deal
    this.result1[index].rate=resObject.rate
    this.result1[index].mrp=resObject.mrp
    this.result1[index].free=resObject.free
    this.result1[index].exp=resObject.exp


    // console.log(resObject)
    }
  },
  modifyArray(){
    console.log("here At Modify array")
    var array=this.result
    var resArray=[]
    var last
    var present
    console.log(array[0],array[0].stock)
    var tempSum=array[0].stock
    var deal=array[0].deal
    var free =array[0].free
    var mrp=array[0].mrp
    var rate=array[0].rate
    var expiryDate=array[0].exp
    last=array[0].name
    var sI=0
    var o={}
    var batch=array[0].batch
    o[array[0].name]={}
    o[array[0].name][batch]=[]
    o[array[0].name][batch].push(array[0])
    o[array[0].name]["batches"]=[batch]

    console.log("here",o)
    for(let i=1;i<array.length;i++){
      present=array[i].name
      batch=array[i].batch
      if(present===last){
        
        if(o[array[i].name][batch]===undefined){
          o[array[i].name][batch]=[]
          o[array[i].name]["batches"].push(batch)
          
        }
        o[array[i].name][batch].push(array[i])
      tempSum+=array[i].stock //stock Addition
      if(array[i].deal<deal){
        deal=array[i].deal       //deal comparision
      }
      if(array[i].free<free){
        free=array[i].free    //free comparision
      }
      if(array[i].mrp>mrp){
        mrp=array[i].mrp       //mrp comaprision
      }
      if(array[i].rate>rate){
        rate=array[i].rate     //rate comparision
      }
      expiryDate=this.compareDates(expiryDate,array[i].exp)   //expiry date minimum
    }
    else{
        o[last]["sI"]=sI
        o[last]["batches"].push("All")
        o[last]["All"]={
          name:last,
          batch:o[last]["batches"],
          stock:tempSum,
          deal:deal,
          free:free,
          mrp:mrp,
          rate:rate,
          exp:expiryDate,
          supplier:array[i-1].supplier,
          company:array[i-1].company
        }
        o[last].batchValue="All"
        resArray.push({
          name:last,
          batch:o[last]["batches"],
          stock:tempSum,
          deal:deal,
          free:free,
          mrp:mrp,
          rate:rate,
          exp:expiryDate,
          supplier:array[i-1].supplier,
          company:array[i-1].company
        })
        sI+=1
        expiryDate=array[i].exp
        rate=array[i].rate
        mrp=array[i].rate
        deal=array[i].deal
        free=array[i].free
        tempSum=array[i].stock
        o[array[i].name]={}
        o[array[i].name][batch]=[]
        o[array[i].name][batch].push(array[i])
        o[array[i].name]["batches"]=[batch]
        
      }
      last=present
    }
    o[last]["sI"]=sI
    o[last]["batches"].push("All")
    o[last]["All"]={name:last,
          batch:o[last]["batches"],
          stock:tempSum,
          deal:deal,
          free:free,
          mrp:mrp,
          rate:rate,
          exp:expiryDate,
          supplier:array[array.length-1].supplier,
          company:array[array.length-1].company}
          o[last].batchValue="All"
    resArray.push({
          name:last,
          batch:o[last]["batches"],
          stock:tempSum,
          deal:deal,
          free:free,
          mrp:mrp,
          rate:rate,
          exp:expiryDate,
          supplier:array[array.length-1].supplier,
          company:array[array.length-1].company
        })

      // console.log(resArray)
      console.log("objectNames",this.objectDetails)
      this.objectDetails=o
      console.log("objectNames",this.objectDetails)
      this.result1=resArray
      console.log(this.result1)
      this.resultLength=this.result1.length
  }

  // console.log()
    ,
    selectAllRows(){
      // const page=this.pageNo
      // const page_size=this.pagesize
      // const array=this.result1
      // console.log("array=",this.result1)
      // console.log(`page=${page} & page_size=${page_size}`)
      // count=0
      
      // if(this.statusCheckAllRows===true){
      //   console.log("true")

      // }
      // else{
      //   console.log("false")
      // 

      let i=0
      const checkValue=this.statusCheckAllRows
      const rowsDisplayed=document.getElementsByTagName("tr")
      function check(e){
        e.checked=true
      }
      function deCheck(e){
        e.checked=false
      }
      for (i=1;i<rowsDisplayed.length;i++){
        console.log(rowsDisplayed[i])
        if(checkValue===true){
        rowsDisplayed[i].classList="b-table-row-selected table-active"
        }
        else{
          rowsDisplayed[i].classList=""
        }
        // console.log(rowsDisplayed[i].children[0].children[0].children[0].children[0])
        const tempCheckBox=rowsDisplayed[i].children[0].children[0].children[0].children[0]
        console.log(tempCheckBox)

        tempCheckBox.addEventListener("click",checkValue===true?check(tempCheckBox):deCheck(tempCheckBox))
    }}
    
    ,
    selectDeselectRows(id){
      const valueCheck = this.statusSelectOne

      const indexStart = this.pageNo * 10 - 10
      let row = 1

      for (let i = indexStart; i < indexStart + 10; i++) {
        if (this.result1[i].id === id) {

          break
        }
        row += 1
      }
      const rowsDisplayed = document.getElementsByTagName("tr")
      if (valueCheck === true) {

        rowsDisplayed[row].classList = "b-table-row-selected table-active"
      }
      else {
        rowsDisplayed[row].classList = ""
      }
    
  },
  delselectHeader(){
      this.statusCheckAllRows=false
      this.selectAllRows()
    },
    searchChangeHandler(array,lengthOfArray){
      // var bTable=document.getElementById("table")
      this.result1=array
      this.resultLength=lengthOfArray

      // console.log(this.result,this.result.length)
    },

  },
  
  
  mounted(){
    // this.getList()  // after implementationadfsad
  }
}

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style >
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.buttonContainer{
  height: 100px;
  width: 50%;
  text-align: center;
  margin: auto;
  margin-bottom: 10px;
}
.header{
  text-align: center;
}
.button{
  width: 25%;
  margin: auto;
}
.tableContainer{
  padding: 15px;
  width: 80%;
  margin: auto;
}
.editButtonRow,.deleteButtonRow
{background-color: white;
  border: none;
  padding: none;
  text-align: center;

}
.buttonDeleteAll{
  margin-right: auto;
  border-radius: 50px;
}
.pagination{
  margin: auto;
}
.paginationContainer{
  display: flex;
  flex-direction: row;
}

</style>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>