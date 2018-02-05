<template>
  <div id="app">
    <h1>Research App</h1>
    <div class="row search">
        <div class="col-md-8 col-md-offset-2">
              <input type="text" name="" placeholder="Input your keyword..." v-model="keyword" class="form-control search-box">
              <button type="button" class="btn btn-primary" name="button" v-on:click="getData">Search</button>
              <button type="button" class="btn btn-warning" name="button" v-on:click="exportXlsx">Export to Excel</button>
        </div>
    </div>
    <br>
    <div class="condition row">
      <span><b>Page Start</b></span><input type="number" name="" value="" id="page-start" v-model="pageStart">
      <span><b>Page End</b></span><input type="number" name="" value="" id="page-end" v-model="pageEnd">
      <span><b>Favorite</b></span>
      <span>From</span><input type="number" name="" value="" id="page-end" v-model="lowestFavoriteNumber">
    </div>
    <br>
    <div class="row result">
      <table class="table table-bodered">
        <thead>
          <td><b>Product</b></td>
          <td>
            <b>Favorite Number</b>
            <img src="./assets/images/sort.png" alt="" class="sort-icon" v-on:click="sort"></img>
          </td>
        </thead>
        <tbody>
          <tr v-for="item in data">
            <td class="item-img-name">
              <a v-bind:href="item.link"  target="_blank">
                <!-- <img v-bind:src="item.imgLink" alt=""> -->
                <p>{{item.name}}</p>
              </a>
            </td>
            <td>
              <p>{{ item.favoriteNumber }}</p>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'app',
  data () {
    return {
      keyword: "tshirt",
      pageStart: 1,
      pageEnd: 3,
      lowestFavoriteNumber: 100,
      data: [],
      sortOrder: true,
      pageDone: 0,
      productTags: [],
      products: [],
      numberProducts: 0,
      done: 0
    }
  },
  methods: {
    getData: function(){
      this.resetData();
      var that = this;
      const ONE_TIME_NUMBER_PAGE = 1;
      var numberProducts = 0;
      $.ajax({
  	  		url: "https://www.etsy.com/search?q="+this.keyword+"&as_prefix=ts&explicit=1&page=1",
          dataType: 'html',

  	  	}).then(
          function(res){
            var lastPage = parseInt($(res).find('.ss-icon.ss-navigateright.icon-smaller-lg.icon-t-1').closest('a').prev('a').find('span').eq(1).text());
            var done = 0; // done check number favorite
            for(let pageIndex = that.pageStart; pageIndex <= that.pageEnd; pageIndex++) {
              that.getProductsOfPage(pageIndex);
            }
        })
      },
      exportXlsx: function() {
        var opts = [{sheetid:'Sheet One',header:true}];
        var result = alasql('SELECT * INTO XLSX("'+this.keyword+'_page_from'+this.pageStart+"-"+this.pageEnd+'_favorite_'+this.lowestFavoriteNumber+'.xlsx",?) FROM ?',
                          [opts,[this.data]]);
      },
      removeDuplicateUsingSet: function(arr){
          let unique_array = Array.from(new Set(arr));
          return unique_array;
      },
      getProductsOfPage(pageIndex){
        var that = this;
        $.ajax({
              url: "https://www.etsy.com/search?q="+that.keyword+"&as_prefix=ts&explicit=1&page="+pageIndex,
              dataType: 'html',
            }).then(
              function(res){
                var products = $(res).find('.search-listings-group div.v2-listing-card');
                that.productTags.push.apply(that.productTags, products);
                that.pageDone++;

                // if DONE crawl all product -> filter to remove dupplicate product
                if((that.pageDone == that.pageEnd - that.pageStart + 1)) {
                  that.filterUniqueListProducts();

                  // get Infor for each product
                  that.numberProducts += that.products.length;
                  for (let i = 0; i<that.products.length; i++){
                    that.myAjax(i);
                  }
                }
              }
          )
      },
      resetData: function()  {
        this.data = [];
        this.sortOrder = true;
        this.pageDone = 0;
        this.productTags = [];
        this.products = [];
        this.numberProducts = 0;
        this.done = 0;
      },
      filterUniqueListProducts: function() {
        var listIds = this.productTags.map(function(product){return parseInt($(product).attr('data-listing-id'))});
        var uniqueListIds = this.removeDuplicateUsingSet(listIds);
        for(let i = 0; i<uniqueListIds.length; i++) {
          var product = this.productTags.filter(function(product){return parseInt($(product).attr('data-listing-id')) == uniqueListIds[i]})[0];
          this.products.push(product);
        }
      },
      myAjax: function(index) {
        var that = this;
        var product = this.products[index];
        $.ajax({
            url: $(product).find('a').attr('href'),
            dataType: 'html',
            success: function(res){
              //var imgLink = $(product).find('.v2-listing-card__img').find('img').attr("src");
              var link = $(product).find('a').attr('href');
              //var bestseller = $(product).find("p:contains('Bestseller')");
              var name = $(product).find('.v2-listing-card__info').find('p').first().text().trim();
              var favoriteTag = $(res).find('.properties').find("li:contains('Favorited')").find('a');
              if(favoriteTag !== undefined) {
                var favoriteNumber = parseInt($(favoriteTag).text().replace(/\D/g,'') || "0");
              } else favoriteNumber = 0;
              if(favoriteNumber >= that.lowestFavoriteNumber) {
                that.data.push({
                    'name': name,
                    //'imgLink': imgLink,
                    'link': link,
                    //'bestseller': bestseller,
                    'favoriteNumber': favoriteNumber
                });
              }
              that.done += 1;
              if(that.done == that.numberProducts) {
                that.sort();
                //alert("Done");
              }
            },
            error: function(err) {
              console.log(err);
            }
          })
      },
      sort: function() {
        this.sortOrder = !this.sortOrder;
        if(this.sortOrder) {
          this.data.sort((a, b) => {
            return a.favoriteNumber - b.favoriteNumber;
          });
        } else {
          this.data.sort((a, b) => {
            return b.favoriteNumber - a.favoriteNumber;
          });
        }
      }
    }

  }
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
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

.search-box {
    width: 60% !important;
    display: inline-block;
}
.item-img-name {
  img {
    float: left;
    width: 200px;
    height: 200px;
    margin-left: 20px;
  }
}
.sort-icon {
    width: 20px !important;
    height: 20px !important;
    display: inline-block;
}
</style>
