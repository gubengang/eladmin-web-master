<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">文章标题</label>
        <el-input v-model="query.title" clearable placeholder="文章标题" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.created"
          start-placeholder="createdStart"
          end-placeholder="createdStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="ID">
            <el-input v-model="form.id" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="用户ID" prop="userId">
            <el-input v-model="form.userId" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="文章标题" prop="title">
            <el-input v-model="form.title" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="描述" prop="description">
            <el-input v-model="form.description" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="内容">
            <el-input v-model="form.content" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="created" prop="created">
            <el-input v-model="form.created" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="status">
            <el-input v-model="form.status" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="id" label="ID" />
        <el-table-column prop="userId" label="用户ID" />
        <el-table-column prop="title" label="文章标题" />
        <el-table-column prop="description" label="描述" />
        <el-table-column prop="content" label="内容" />
        <el-table-column prop="created" label="created" />
        <el-table-column prop="status" label="status" />
        <el-table-column v-if="checkPer(['admin','mBlog:edit','mBlog:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudMBlog from '@/api/mBlog'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, userId: null, title: null, description: null, content: null, created: null, status: null }
export default {
  name: 'MBlog',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  cruds() {
    return CRUD({ title: '博客', url: 'api/mBlog', idField: 'id', sort: 'id,desc', crudMethod: { ...crudMBlog }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'mBlog:add'],
        edit: ['admin', 'mBlog:edit'],
        del: ['admin', 'mBlog:del']
      },
      rules: {
        userId: [
          { required: true, message: '用户ID不能为空', trigger: 'blur' }
        ],
        title: [
          { required: true, message: '文章标题不能为空', trigger: 'blur' }
        ],
        description: [
          { required: true, message: '描述不能为空', trigger: 'blur' }
        ],
        created: [
          { required: true, message: '不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'title', display_name: '文章标题' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
