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
      <el-row>
        <el-alert
          title="添加商品信息"
          type="info"
          center
          show-icon
          closeable="false"
        ></el-alert>
      </el-row>
      <el-row>
        <el-steps
          :space="200"
          :active="activeIndex - 0"
          finish-status="success"
          align-center
        >
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
      </el-row>
      <el-row>
        <el-form
          label-position="top"
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="100px"
        >
          <el-tabs
            tab-position="left"
            v-model="activeIndex"
            :before-leave="beforeTabLeave"
            @tab-click="tabClicked"
          >
            <el-tab-pane label="基本信息" name="0">
              <el-form-item label="活动名称" prop="goods_name">
                <el-input v-model="addForm.goods_name"></el-input>
              </el-form-item>
              <el-form-item label="商品价格" prop="goods_price">
                <el-input
                  v-model="addForm.goods_price"
                  type="number"
                ></el-input>
              </el-form-item>
              <el-form-item label="商品重量" prop="goods_weight">
                <el-input
                  v-model="addForm.goods_weight"
                  type="number"
                ></el-input>
              </el-form-item>
              <el-form-item label="商品数量" prop="goods_number">
                <el-input
                  v-model="addForm.goods_number"
                  type="number"
                ></el-input>
              </el-form-item>
              <el-form-item label="商品分类" prop="goods_cat">
                <el-cascader
                  v-model="addForm.goods_cat"
                  :options="goodslist"
                  :props="goodsprops"
                  @change="handleChange"
                  clearable
                ></el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品参数" name="1">
              <el-form-item
                :label="item.attr_name"
                v-for="item in manyData"
                :key="item.attr_id"
              >
                <el-checkbox-group v-model="item.vals">
                  <el-checkbox
                    :label="cd"
                    v-for="(cd, i) in item.attr_vals"
                    :key="i"
                    border
                  ></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品属性" name="2">
              <el-form-item
                :label="item.attr_name"
                v-for="item in onlyData"
                :key="item.attr_id"
              >
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane label="商品图片" name="3">
              <el-upload
                :action="uploadurl"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture"
                :headers="hearderobj"
                :on-success="handlesuccess"
              >
                <el-button size="small" type="primary">点击上传</el-button>
              </el-upload>
              <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
                <img :src="imgurl" alt="" />
              </el-dialog>
            </el-tab-pane>
            <el-tab-pane label="商品内容" name="4">
              <quill-editor
                ref="myQuillEditor"
                v-model="addForm.goods_introduce"
              />
              <el-button type="primary" @click="add">添加商品</el-button>
            </el-tab-pane>
          </el-tabs>
        </el-form>
      </el-row>
    </el-card>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: {
          required: true,
          message: '请输入商品名称',
          trigger: 'blur'
        },
        goods_price: {
          required: true,
          message: '请输入商品价格',
          trigger: 'blur'
        },
        goods_weight: {
          required: true,
          message: '请输入商品重量',
          trigger: 'blur'
        },
        goods_number: {
          required: true,
          message: '请输入商品数量',
          trigger: 'blur'
        },
        goods_cat: {
          required: true,
          message: '请选择商品分类',
          trigger: 'blur'
        }
      },
      selectkeys: [],
      goodslist: [],
      goodsprops: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        expandTrigger: 'hover'
      },
      manyData: [],
      onlyData: [],
      uploadurl: 'http://127.0.0.1:8888/api/private/v1/upload',
      hearderobj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      dialogVisible: false,
      imgurl: ''
    }
  },
  methods: {
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
        return true
      }
    },
    async getgoodslist() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品类表数据失败')
      }
      this.goodslist = res.data
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.info('请选择商品分类')
        return false
      }
    },
    async tabClicked() {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'many'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取分类参数失败')
        }
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.trim() === '' ? [] : item.attr_vals.split(' ')
          item.vals = item.attr_vals
        })
        console.log(res)
        this.manyData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'only'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取分类参数失败')
        }
        console.log(res)
        this.onlyData = res.data
      }
    },
    handlePreview(file) {
      this.dialogVisible = true
      this.imgurl = file.response.data.url
    },
    handleRemove(file) {
      const { picinfo } = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex((x) => x.pic === picinfo)
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm.pics)
    },
    handlesuccess(response) {
      var picinfo = {
        pic: response.data.tmp_path
      }
      this.addForm.pics.push(picinfo)
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.info('请填写相关信息')
        }
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        this.manyData.forEach((item) => {
          item.attr_value = item.attr_vals.join(' ')
          const vals = {
            attr_id: item.attr_id,
            attr_value: item.attr_value
          }
          form.attrs.push(vals)
        })
        this.onlyData.forEach((item) => {
          const val = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          form.attrs.push(val)
        })
        const { data: res } = await this.$http.post('goods', form)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  created() {
    this.getgoodslist()
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
}
.el-checkbox {
  margin: 10px;
}
img {
  width: 100%;
}
.el-button {
  margin-top: 20px;
}
</style>
