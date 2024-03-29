<template>
  <el-dialog :title="title" :visible.sync="Visible" @close="close">
    <div class="content">
      <el-form label-width="80px" :model="form" :rules="rules" ref="ruleForm">
        <div class="base">
          <h4>基础信息</h4>
          <el-form-item label="所属项目" prop="projectId">
            <el-select
              v-model="form.projectId"
              placeholder="请选择"
              class="projectSelect"
            >
              <el-option
                v-for="item in myProjects"
                :key="item.id"
                :label="item.groupName + '/' + item.projectName"
                :value="item.id"
              >
                <span style="float: left">{{
                  item.groupName + '/' + item.projectName
                }}</span>
              </el-option>
            </el-select>
          </el-form-item>
          <el-row type="flex" :gutter="0">
            <el-col :span="24">
              <el-form-item label="页面标题" prop="name">
                <el-input v-model="form.name"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row type="flex" :gutter="0">
            <el-col :span="24">
              <el-form-item label="语言" prop="lang">
                <!-- <el-input v-model="form.lang"></el-input> -->
                <el-select v-model="form.lang">
                  <el-option
                    v-for="l in langs"
                    :key="l"
                    :label="langDisp[l]"
                    :value="l"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-form-item label="描述" prop="desc">
            <el-input
              type="textarea"
              placeholder="该页面的简要描述1-50个字"
              :minlength="1"
              :maxlength="50"
              v-model.trim="form.desc"
            ></el-input>
          </el-form-item>
          <!-- <el-form-item label="标签">
            <tags :can-add="true" :category-id="1" :tags.sync="form.tags"></tags>
          </el-form-item>
          <el-form-item label="设为模板" v-if='type != "fork"'>
            <el-radio-group v-model="form.visibilitylevel">
              <el-radio :label="0">否</el-radio>
              <el-radio :label="1">是</el-radio>
            </el-radio-group>
          </el-form-item> -->
        </div>
      </el-form>
    </div>
    <div slot="footer" class="dialog-footer">
      <el-button @click="close">取消</el-button>
      <el-button type="primary" @click="addPageInfo('ruleForm')"
        >确定添加，编辑详情</el-button
      >
    </div>
  </el-dialog>
</template>
<style lang="stylus" rel="stylesheet/stylus" scoped type="text/stylus">
.content {
  .projectSelect {
    width: 100%;
  }

  .info {
    position: relative;

    .desc {
      padding: 10px;
    }

    .time {
      position: absolute;
      bottom: -15px;
      right: 0;
    }
  }

  .doc {
    max-height: 400px;
    overflow: auto;
  }
}
</style>
<script type="text/ecmascript-6">
import BaseDialog from 'src/extend/BaseDialog'
import Server from 'src/extend/Server'
import Tags from 'src/components/Tags'

export default {
  mixins: [BaseDialog],
  name: 'DCopyPage',
  components: { Tags },
  data() {
    return {
      Visible: true,
      formLabelWidth: '150px',
      myProjects: [],
      form: {
        name: '',
        desc: '',
        visibilitylevel: 0,
        projectId: null,
        tags: [],
        type: '0',
        content: '',
        lang: 'en',
      },
      source: {},
      langDisp: {
        zh: '中文',
        en: '英文',
        es: '西班牙',
        de: '德语',
        vi: '越南语',
        fr: '法语',
        ja: '日语',
        nl: '荷兰语',
        pl: '波兰语',
        ko: '韩语',
        ru: '俄罗斯',
        it: '意大利',
        tr: '土耳其',
        zhhant: '繁体中文'
      },
      langs: [
        'zh',
        'en',
        'es',
        'de',
        'vi',
        'fr',
        'ja',
        'nl',
        'pl',
        'ko',
        'ru',
        'it',
        'tr',
        'zhhant'
      ],
      rules: {
        projectId: [
          { required: true, message: '请选择所属项目', trigger: 'change' },
        ],
        name: [{ required: true, message: '请输入页面标题', trigger: 'blur' }],
        lang: [{ required: true, message: '请输入语言', trigger: 'blur' }],
        desc: [{ required: false, message: '请输入描述内容', trigger: 'blur' }],
      },
    }
  },
  watch: {},
  mounted() {
    this.getGrops()
    this.form.visibilitylevel = 0
  },
  methods: {
    getGrops: function () {
      Server({
        url: 'project/project',
        method: 'get',
        needLoading: true,
        params: {
          count: 100,
          start: 0,
        },
      })
        .then((response) => {
          var data = response.data.data
          this.myProjects = data
          if (data.length == 0) {
            // 如果没有项目。提示前往添加项目
            this.$confirm(
              '你还没有任何项目，添加页面前需要先添加项目：前往添加项目',
              '提示',
              {
                type: 'warning',
              }
            )
              .then(() => {
                this.close()
                this.$router.push({ path: '/projects/new' })
              })
              .catch(() => {
                this.$message('已取消')
              })
          }
        })
        .catch(() => {})
    },
    updateFork(data) {
      Server({
        url: 'editor/pages/update-fork',
        method: 'post',
        trimNull: false,
        data: {
          projectId: this.source.projectId,
          id: this.source.id,
        },
      }).then((_) => {
        this.ema.fire('dashboard_featured.pageAddSuccess')
      })
    },
    addPageInfo(ruleForm) {
      let me = this
      this.$refs[ruleForm].validate((valid) => {
        if (valid) {
          Server({
            url: 'editor/pages/save',
            method: 'post',
            needLoading: true,
            trimNull: false,
            data: me.form,
          }).then((res) => {
            me.$message({
              showClose: true,
              message: '添加成功',
              type: 'success',
            })
            if (me.type === 'fork') {
              this.updateFork()
              this.$router.push({
                path: '/project',
                query: { id: me.form.projectId },
              })
            }
            // this.ema.fire('projects_cdoc.pageAddSuccess')
            me.close()
            Server({
              url: 'editor/pages/info',
              method: 'post',
              needLoading: false,
              data: {
                id: res.data.data.id + '',
              },
            }).then((res) => {
              Server({
                url: 'editor/pages/save',
                method: 'post',
                needLoading: false,
                data: {
                  ...res.data.data,
                  key: me.form.keypre + 'LanG' + me.form.lang,
                },
              }).then(() => {
                this.ema.fire('projects_cdoc.pageAddSuccess')
              })
            })
          })
        }
      })
    },
  },
}
</script>

