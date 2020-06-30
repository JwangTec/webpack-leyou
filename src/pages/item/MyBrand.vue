<template>
  <v-card>
    <v-card-title>
      <v-btn color="primary" @click="addBrand">新增品牌</v-btn>

      <!--弹出的对话框-->
      <v-dialog max-width="500" v-model="show" persistent>
        <v-card>
          <!--对话框的标题-->
          <v-toolbar dense dark color="primary">
            <v-toolbar-title>{{ isEdit ? "修改" : "新增" }}品牌</v-toolbar-title>
            <v-spacer/>
            <!--关闭窗口的按钮-->
            <v-btn icon @click="closeWindow"><v-icon>close</v-icon></v-btn>
          </v-toolbar>

          <!--对话框的内容，表单-->
          <v-card-text class="px-5">
<!--            使用局部组件-->
            <MyBrandForm @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"/>
<!--            <brand-form/>-->
          </v-card-text>
        </v-card>
      </v-dialog>


      <v-spacer/>
      <v-text-field label="输入关键字搜索" v-model="key" append-icon="search" hide-details single-line/>
    </v-card-title>

    <v-divider>
    </v-divider>

    <v-data-table
      :headers="headers"
      :items="brands"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center">
          <img v-if="!!props.item.image" width="102" height="36" :src="props.item.image"/>
        </td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="justify-center layout">
          <v-btn color="info" @click="editBrand(props.item)">编辑</v-btn>
          <v-btn color="warning" @click="deleteBrand(props.item)">删除</v-btn>
        </td>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
  //导入局部组件处理表单
  import MyBrandForm from "./MyBrandForm";
  export default {
    name: "MyBrand",
    components: {MyBrandForm},
    data() {
      return {
        headers: [
          {text: '品牌id', align: 'center', sortable: true, value: 'id'},
          {text: "品牌名称", value: "name", sortable: false, align: "center"},
          {text: "品牌logo", value: "image", sortable: false, align: "center"},
          {text: "首字母", value: "letter", sortable: true, align: "center"},
          {text: "操作", value: "letter", sortable: false, align: "center"}
        ],
        brands: [],
        pagination: {},
        totalBrands: 0,
        loading: false,
        key: "", //查询文本框内容获取
        show: false, //对话框显示
        oldBrand: {}, //获取现在点击的品牌数据
        oldId: 0, //获取点击时的id
        isEdit: false  //判断新增还是修改
      }
    },
    //自定义数据，或者调用的方法
    created() {
      // this.brands=[
      //   {id: 1 , name:"小米", image:"1.jpg", letter:"O"},
      //   {id: 2 , name:"小", image:"2.jpg", letter:"F"}
      // ];
      // this.totalBrands=10;
      //搜索框功能
      this.loadBrands();
    },

    //监控
    watch: {
      //搜索框内容监控,强制将其页数为1
      key() {
        this.pagination.page = 1;
        this.loadBrands();
      },
      //复杂类型：深度监控, 如果pagination发生改变，就会调用里面的方法
      pagination: {
        deep: true,
        handler() {
          this.loadBrands();
        }
      }
    },
    methods: {
      //搜索文本框数据后台查询
      loadBrands() {
        this.loading = true,
          this.$http.get("/item/brand/page", {
            params: {
              //获取数据后获取pagination的当前页，每页大小，排序方式，是否降序
              page: this.pagination.page,
              rows: this.pagination.rowsPerPage,
              sortBy: this.pagination.sortBy,
              desc: this.pagination.descending,
              //搜索文本框内容条件
              key: this.key
            }
          }).then(resp => {
            this.brands = resp.data.items;
            this.totalBrands = resp.data.total;
            console.log(resp);
            this.loading = false;
          })
      },

      addBrand() {
        //控制弹窗可见
        this.isEdit = false;
        this.show = true;
        this.oldBrand = null;
      },

      closeWindow() {
        this.show = false;

        //重新加载数据
        this.loading = false;
      },

      editBrand(oldBrand) {
        // this.show = true;
        // //获取要编辑的Brand
        // this.oldBrand = oldBrand;
        this.$http.get("/item/category/bid/" + oldBrand.id)
          .then(({data}) => {
            this.isEdit = true;
            this.show = true;
            this.oldBrand = oldBrand
            this.oldBrand.categories = data;
          })
      },

      deleteBrand(item) {
        this.$message.confirm('此操作将永久删除该品牌, 是否继续?').then(() => {
          // 发起删除请求
          this.$http.delete("/item/brand/delete?bid=" + item.id,)
            .then(() => {
              // 删除成功，重新加载数据
              this.$message.success("删除成功！");
              this.loadBrands();
            })
        }).catch(() => {
          this.$message.info("删除已取消！");
        });
      }
    },

    //指定局部组件
    comments: {
      MyBrandForm
    }
  }
</script>

<style scoped>

</style>
