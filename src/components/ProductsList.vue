<template>
  <div class="container-fluid">
    <div class="row">
      <div>
        <b-form-group label-for="input-formatter" description="Pesquise pelo nome do produto" class="mb-0">
          <label>Pesquisar Produto</label>
          <b-form-input v-model="searchProd" class="w-50 mx-auto"
            placeholder="Pesquise pelo produto desejado" @input="pageReset"></b-form-input>

        </b-form-group>
      </div>
      <div class="col-4 mt-4" v-for="product in showProds" :key="product.id">
        <b-card>

          <img :src="product.src" class="custom-img-size">
          <template>
            <h6 class="mb-0 mt-3"><b>{{ product.name }}</b></h6>
          </template>

          <b-card-body>
            <b-card-text>
              {{ priceFormat(product.cost_price) }}
            </b-card-text>
          </b-card-body>

          <li v-for="(variant, variantIndex) in product.variants" :key="variantIndex">
            Tam: {{ variant.value }}
          </li>

          <!-- <b-card-body> -->
          <div class="card-prods-category">
            <p v-for="(category, categoryIndex) in product.categories" class="card-text" :key="categoryIndex">
              {{ category.name }}
            </p>
          </div>
          <!-- </b-card-body> -->
        </b-card>
      </div>
    </div>

    <nav aria-label="Page navigation">
      <ul class="pagination justify-content-center mt-4">
        <li class="page-item">
          <a href="#" class="page-link" @click.prevent="pageChange(pageCurrent - 1)"
            :disabled="pageCurrent === 1">Anterior</a>
        </li>
        <li class="page-item" v-for="pageNumb in currentPage" :key="pageNumb"
          :class="{ active: pageCurrent === pageNumb || pageNumb === '...' }">
          <a href="#" class="page-link" @click.prevent="pageChange(pageNumb)">{{ pageNumb }}</a>
        </li>
        <li class="page-item">
          <a href="#" class="page-link" @click.prevent="pageChange(pageCurrent + 1)"
            :disabled="pageCurrent === countedPages">Próxima</a>
        </li>
      </ul>
    </nav>
  </div>

</template>

<script>
import Axios from 'axios';

export default {
  name: 'ProductsList',
  data() {
    return {
      products: [],
      searchProd: '',
      prodsPerPage: 10,
      pageCurrent: 1
    };
  },
  computed: {
    filterProds() {
      return this.products.filter(product =>
        product.name.toLowerCase().includes(this.searchProd.toLowerCase())
      );
    },
    showProds() {
      const start = (this.pageCurrent - 1) * this.prodsPerPage;
      const end = start + this.prodsPerPage;
      return this.filterProds.slice(start, end);
    },
    countedPages() {
      return Math.ceil(this.filterProds.length / this.prodsPerPage);
    },
    currentPage() {
      let numbersPage = [];
      if (this.countedPages <= 7) {
        for (let i = 1; i <= this.countedPages; i++) {
          numbersPage.push(i);
        }
      } else {
        if (this.pageCurrent <= 4) {
          numbersPage = [1, 2, 3, 4, "...", this.countedPages];
        } else if (this.pageCurrent >= this.countedPages - 3) {
          numbersPage = [1, "...", this.countedPages - 3, this.countedPages - 2, this.countedPages - 1, this.countedPages];
        } else {
          numbersPage = [1, "...", this.pageCurrent - 1, this.pageCurrent, this.pageCurrent + 1, "...", this.countedPages];
        }
      }
      return numbersPage;
    }
  },
  methods: {
    getProducts() {
      Axios.post('https://apibgdev.nswebservice.com.br/store/10/products', {
        limit: 100,
      })
        .then((res) => {
          this.products = res.data;
          console.log(this.products);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    pageChange(page) {
      if (page === '...') return;
      if (page >= 1 && page <= this.countedPages) {
        this.pageCurrent = page;
      }
    },
    priceFormat(price) {

      if (typeof price !== 'number') {
        return price;
      }
      
      return price.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
    },

    pageReset(){
      this.pageCurrent = 1;
    }
  },
  mounted() {
    this.getProducts();
  }
}
</script>

<style scoped>
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

.custom-img-size {
  width: 200px !important;
  height: 200px !important;
}

.card-prods-category {
  display: flex;
  justify-content: center;
  align-items: center;
}

.card-prods-category .card-text {
  margin-left: 20px;
  margin-top: 10px;
  font-size: 12px;
}

/* .card-prods-category .card-text:last-child{
  margin-top:0;
} */
</style>
