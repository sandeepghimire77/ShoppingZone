<template>
 <div>

    
    
    
    
 <header class="masthead text-white text-center">
      
      <div class="container p-5">
        <div class="row">
          <div class="col-xl-12 mx-auto">
            
          </div>
          <div class="col-md-10 col-lg-8 col-xl-7 mx-auto">
            <form>
              <div class="form-row">
                <select class="col-4 col-md-3 form-control" style="height: 48px; font-weight:bolder; font-size: 18px" v-model="selectedCategory">
                  <option v-for="category in categories" v-bind:key="category.id" v-bind:value="category.id">{{category.name}}</option>
                </select>
                <div class="col-7 col-md-8 mb-2 mb-md-0 form-group text-left">
                  <input type="text" class="form-control form-control-lg" v-bind:class="{ 'is-invalid': !$v.term.required && $v.term.$dirty}" placeholder="Enter your item name here..." v-model="term" @input="$v.term.$touch()" v-on:keyup.enter="search()" required>
                  <span class="text-warning" v-if="!$v.term.required && $v.term.$dirty">search term can not be empty</span>
                </div>
                <div class="col-xs-10 col-sm-10 col-1 col-md-1">
                  <button type="button" class="btn btn-block btn-lg btn-dark text-light fa fa-search" style="padding: 0.5rem;" @click="search()"></button>
                </div>
              </div> 
            </form>
          </div>
        </div>
      </div>
    </header>


<div class="container">
    <gmap-map   :center="{lat:54.310027, lng:10.131051}"
  :zoom="7"
  map-type-id="terrain"
  style="100%; height: 600px"
>
<gmap-marker
      :key="index"
      v-for="(m, index) in markers"
      :position="m.position"
      :clickable="true"
      :draggable="true"
      @click="center=m.position"
    ></gmap-marker>
</gmap-map>
</div>

    
    <div class="container">
      <div class="row">
        <div class="card m-4" style="width: 20rem;" v-for="product in products" v-bind:key="product.id">
  <img class="card-img-top" :src="getImageAddress(product.image)" alt="Card image cap">
  <div class="card-block">
    <h4 class="card-title text-info m-2">{{product.name}}</h4>
    <p class="card-text text-dark m-2" style="max-height: 40px; min-height:40px">{{product.description.slice(0, 100) + "..."}}</p>
    <router-link class="btn btn-dark m-2 float-right text-light" :to="'/productdetail/' + product.id">more</router-link>                    
  </div>
</div>



  
  </div>
    </div>

 
 
</div>
    


  



</template>

<script>
import { required } from 'vuelidate/lib/validators'
import { diff } from 'semver';
export default {
  name: 'Search',
  data () {
    return {
        categories: [],
        products: [],
        selectedCategory: -1,
        term: "",
        size: 10,
        start: 0,
        markers: []
    }
  },
  validations : {
    term: {
      required   
    }
  },
  methods: {
    search: function(){
      if(this.term != "")
      {
      if(!this.$v.$invalid){
        let categoryId = -1;
        if(this.selectedCategory != (this.categories.find(e=> e.name == "All").id))
          categoryId = this.selectedCategory;
        
        
        this.$router.push(`/search?key=${this.term}&cid=${categoryId}&size=${9}&start=${0}`);
        this.$toasted.show('searching...');
      }
      }
    },
    getImageAddress: function(path){
      
      return this.$gc.getBaseUrl(path)
    },
    init: function(value){        
        if(value.term != "")
        {
        this.categoryId = value.cid;
        this.size = value.size;
        this.start = value.start;
        this.term = value.key;
            let t = this;            
            let url = this.$gc.getBaseUrl("products") + `?key=${this.term}&cid=${this.categoryId}&size=${this.size}&start=${this.start}`;
            console.log(url);
            this.axios.get(url, { headers: this.$auth.AH() })
                .then(function(data){
                    t.products = data.data.products;
                    t.markers = [];
                    console.log(t.products)
                    t.products.forEach(element => {
                      if(element.lat != "" && element.lng != ""){
                        console.log(element)
                        t.markers.push({
                          position: {
                            lat: parseFloat(element.lat),
                            lng: parseFloat(element.lng)
                          }
                        });                      
                    }
                    });
          })
          .catch(function(error){
            
          })
        }
    }
  },
  beforeMount: function() {          
    this.init(this.$router.history.current.query);
    let those=this;        
    those.categories = those.$gc.getItemByKey("categories");
            those.selectedCategory = those.categories.find(e=> e.name == "All").id;
          this.axios.get(this.$gc.getBaseUrl("categories"), { headers: this.$auth.AH() })
          .then(function(data){
            those.$gc.saveItemByKey("categories", JSON.stringify(data.data.categories));
            those.categories = data.data.categories;
            those.selectedCategory = those.categories.find(e=> e.name == "All").id;
          })
          .catch(function(error){
            those.categories = those.$gc.getItemByKey("categories");
            those.selectedCategory = those.categories.find(e=> e.name == "All").id;
          })
  },
  watch: {
        '$route': function(value) {
            this.init(value.query);            
        }
    }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
body {
  font-family: 'Lato', 'Helvetica Neue', Helvetica, Arial, sans-serif; }

h1,
h2,
h3,
h4,
h5,
h6 {
  font-family: 'Lato', 'Helvetica Neue', Helvetica, Arial, sans-serif;
  font-weight: 700; }



</style>
