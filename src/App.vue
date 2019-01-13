<template>
  <div class="wrapper" >

    <p><input type="text" v-model="titleFilter" @input="filterParam"/></p>
    <table	class="table table-bordered">
      <thead>
      <tr>
        <th scope="col"
            v-for="items in header "
            >
          <a href="#" @click="test(items)">{{items.type}}</a>
        </th>
      </tr>
      </thead>
      <div class="preloader" v-show="loading">
        <img src="../data/spinner.gif" alt="">
      </div>
      <tbody>
      <tr v-for="items in paginatedData ">
        <th class="th-id">{{items.id}}</th>
        <th class="th-title">{{items.title}}</th>
        <th class="th-price">{{items.price}}</th>
        <th class="th-color">{{items.color}}</th>
        <th>{{items.department}}</th>
      </tr>
      </tbody>

    </table>
    <button class="btn btn-primary"
      :disabled="pageNumber === 0"
      @click="prevPage">
      < Previous
    </button>
    <button class="btn btn-primary"
      :disabled="pageNumber >= pageCount -1"
      @click="nextPage">
      Next >
    </button>

  </div>
</template>

<script>

import db from '../data/db.json'
import axios from 'axios'
import VueAxios from 'vue-axios'
import Vue from 'vue'

Vue.use(VueAxios, axios)



export default {
  data () {
    return {
      loading: true,
      db: [],
      sortParam: '',
      titleFilter: '',
      pageNumber: 0,
      dbOld: {},
      currentFilter: {
        order: 'asc',
        type: 'id'
      },
      prevFilter: {
        order: 'asc',
        type: 'id'
      },
      toggleFilter: true,
      header: [
        {
          type: 'id'
        },
        {
          type: 'title'
        },
        {
          type: 'price'
        },
        {
          type: 'color'
        },
        {
          type: 'department'
        },
      ]
    }
  },
  props: {
    size:{
      type:Number,
      required:false,
      default: 10
    }
  },
  methods: {
      filterParam(){
        this.pageNumber = 0
      },
      nextPage(){
        this.pageNumber++;
      },
      prevPage(){
        this.pageNumber--;
      },
    test(data){
      if(data.type===this.currentFilter.type && this.currentFilter.order === 'desc'){
        this.currentFilter = {...this.prevFilter}
      }else if(data.type===this.currentFilter.type){
        this.currentFilter.order = 'desc';
      }else{
        this.prevFilter = {...this.currentFilter}
        this.currentFilter.order = 'asc';
        this.currentFilter.type = data.type;
      }
      this.toggleFilter = !this.toggleFilter;
    },
    getData(){
      axios
        .get('../data/db.json')
        .then(response => (this.db = response.data));
        this.loading  = false;
    }
  },
  watch: {
    toggleFilter() {
      if(this.currentFilter.type === "price"){
        this.db.sort((d1, d2) =>  Math.round(d1[this.currentFilter.type]) >  Math.round(d2[this.currentFilter.type]) ? 1 : -1)
      }else{
        this.db.sort((d1, d2) =>  d1[this.currentFilter.type] >  d2[this.currentFilter.type] ? 1 : -1)
      }


      return this.currentFilter.order === 'asc' ? [...this.db] : [...this.db.reverse()]
    }
  },

  computed: {
    filteredList() {
      var tFiltr = this.titleFilter.toLowerCase();
      return this.db.filter(function (elem) {
        if(tFiltr==='') return true;
        else return elem.title.toLowerCase().indexOf(tFiltr) > -1;
      });
    },
    pageCount(){
      let l = this.filteredList.length,
        s = this.size;
      return Math.floor(l/s);
    },
    paginatedData(){
      const start = this.pageNumber * this.size,
        end = start + this.size;
      return this.filteredList
        .slice(start, end);
    }

  },
  mounted() {
    setTimeout(this.getData, 2000);

  }
}





</script>

<style>
  .wrapper{
    padding: 15px;
    max-width: 800px;
    margin: 0 auto;
  }
  .preloader{
    height: 570px;
  }
  .preloader img{
    position: absolute;
    top: 40%;
    left: 50%;
    margin-right: -50%;
    transform: translate(-50%, -50%);
  }
  .table>tbody>tr>th{
    line-height: 40px;
  }
  th{
    width: 10%;
  }
  .th-title{
    width: 60%;
  }
  .th-color{
    width: 30%;
  }

</style>
