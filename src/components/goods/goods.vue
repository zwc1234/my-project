<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="9">
          <el-input
            placeholder="请输入内容"
            v-model="queryinfo.query"
            class="input-with-select"
            clearable
            @clear="getgoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getgoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="addcate">添加商品</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-table :data="goodsList" style="width: 100%" stripe border>
          <el-table-column type="index" label="#"> </el-table-column>
          <el-table-column prop="goods_name" label="商品名称">
          </el-table-column>
          <el-table-column prop="goods_price" label="商品价格(元)" width="80">
          </el-table-column>
          <el-table-column prop="goods_weight" label="商品重量" width="80">
          </el-table-column>
          <el-table-column label="创建时间" width="150">
            <template slot-scope="scope">
              {{ scope.row.add_time  | dateFormat }}
            </template>
          </el-table-column>
          <el-table-column prop="name" label="操作" width="120">
            <template slot-scope="scope">
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
              ></el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="removeGoods(scope.row)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-row>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryinfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryinfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      goodsList: [],
      queryinfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: null
    }
  },
  created() {
    this.getgoodsList()
  },
  methods: {
    async getgoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryinfo
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.total = res.data.total
      this.goodsList = res.data.goods
    },
    handleSizeChange(newsize) {
      this.queryinfo.pagesize = newsize
      this.getgoodsList()
    },
    handleCurrentChange(newnum) {
      this.queryinfo.pagenum = newnum
      this.getgoodsList()
    },
    addcate() {
      this.$router.push('/goods/add')
    },
    async removeGoods(row) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('goods/' + row.goods_id)
      console.log(res)
      console.log(row.goods_id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getgoodsList()
    }
  }
}
</script>
<style lang="less" scoped>
</style>
