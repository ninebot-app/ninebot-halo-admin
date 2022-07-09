<template>
  <div class="content">
    <div class="project-home-panel text-center">
      <div class="container-fluid container-limited"></div>
    </div>
    <!--页面列表信息-->
    <div v-if="!hasData">
      <div class="blank-state">
        <div class="blank-state-icon">
          <i class="ifont icon-empty"></i>
          <span>暂无页面信息，请先添加页面</span>
        </div>
      </div>
      <el-alert class="tipWarp" title="项目介绍" type="info">
        <div v-html="projectMd"></div>
      </el-alert>
    </div>
    <div class="pages clearfix" v-else>
      <div v-show="filterKey">
        <el-button type="info" @click="filterKey = ''">返回</el-button>
        <el-table :data="productListPFilterByKey" border>
          <el-table-column label="标题" prop="name"></el-table-column>
          <!-- <el-table-column label="页面key" prop="keypre"></el-table-column> -->
          <el-table-column label="语言" prop="lang"></el-table-column>
          <el-table-column label="更新时间" prop="updateTime"></el-table-column>
          <el-table-column fixed="right" label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="changeStatus(scope.row)"
                >启用</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-table
        v-show="!filterKey"
        :data="productListPFilterByZh"
        border
        @row-click="(row) => (filterKey = row.keypre)"
      >
        <el-table-column label="标题" prop="name"></el-table-column>
        <el-table-column label="描述" prop="desc"></el-table-column>
        <!-- <el-table-column label="页面key" prop="keypre"></el-table-column> -->
        <!-- <el-table-column label="语言" prop="lang"></el-table-column> -->
        <el-table-column
          label="创建时间（zh语言）"
          prop="createTime"
        ></el-table-column>
      </el-table>
    </div>
    <!--页面列表信息/-->
  </div>
</template>


<script type="text/ecmascript-6">
import BasePage from 'src/extend/BasePage'
import Server from 'src/extend/Server'
const config = require('src/config')
var projectMd = require('src/assets/tip/help/project.md')

export default {
  mixins: [BasePage],
  components: {},
  name: 'projects_deldoc',
  props: {
    id: {
      // 项目id
      type: String,
    }
  },
  data() {
    return {
      projectKey: '',
      projectName: '',
      projectId: null,
      categoryType: {
        1: '活动类',
      },
      productList: [],
      parameter: {
        key: '', // 项目key
        name: '', // 项目名称
        categoryId: '', // 分类id,
        status: '1', // 项目状态,
      },
      pagination: {
        total: 0, // 总页数
        curr: 1, // 当前页数
        size: 10, // 每页几条
      },
      projectMd: projectMd,
      filterKey: '',
    }
  },
  mounted: function () {
    this.projectId = this.$route.query.id - 0
    this.projectKey = this.$route.query.key
    this.projectName = window.decodeURIComponent(this.$route.query.name || '')
    this.detailInfo()
    this.bindEvent('pageAddSuccess', () => {
      this.detailInfo()
    })
  },
  computed: {
    hasData: function (list) {
      return this.productList.length > 0
    },
    productListP() {
      return this.productList.map((i) => ({
        ...i,
        keypre: i.key.split('LanG')[0],
        lang: i.key.split('LanG')[1] || 'zh',
      }))
    },
    productListPFilterByZh() {
      return this.productListP.filter((i) => i.lang === 'zh')
    },
    productListPFilterByKey() {
      return this.productListP.filter((i) => i.keypre === this.filterKey)
    },
  },
  methods: {
    // 1启用、2禁用 0删除
    changeStatus: function (item) {
      var me = this
      me.status = item.status == 1 ? 2 : 1
      var isStatus = ''
      me.status == 2 ? (isStatus = '禁用') : (isStatus = '启用')
      me.$confirm(`确定要${isStatus}吗？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        Server({
          url: 'editor/pages/change-status',
          data: {
            id: +item.id,
            status: +me.status,
          },
          needLoading: true,
          trimNull: false,
          method: 'post',
        }).then((res) => {
          me.detailInfo()
          me.$message({
            type: 'info',
            message: isStatus + '成功！',
          })
        })
      })
    },
    // 详情页面挂载
    detailInfo() {
      Server({
        url: 'editor/pages/list',
        method: 'post',
        needLoading: true,
        trimNull: false,
        data: {
          projectId: this.projectId,
          status: 0
        },
      }).then((res) => {
        this.productList = res.data.data || []
      })
    }
  },
}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped type="text/stylus">
.project-home-panel {
  margin-bottom: 30px;
}

.project-home-environment {
  width: 500px;
  margin: auto;
}

.project-home-url {
  width: 500px;
  margin: auto;
}

.blank-state-icon {
  span {
    font-size: 24px;
  }
}

.pathWarp {
  position: relative;

  i {
    position: absolute;
    bottom: -6px;
    right: -15px;
    color: #20597e;
  }
}

.project-avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  overflow: hidden;
  margin: 20px auto;

  img {
    width: 100%;
    height: 100%;
  }
}

.boxSize {
  position: relative;
  float: left;
  text-align: center;
  background-color: white;
  width: 200px;
  min-height: 240px;
  margin: 0 1em 30px;
  border: 1px solid #ddd;
  overflow: hidden;
  border-radius: 5px;
  cursor: pointer;
  transition: transform 0.4s ease;

  .status {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 999;
    color: #fff;

    span {
      font-size: 12px;
      display: block;
      padding: 0 10px;
      height: 30px;
      line-height: 30px;
      color: #fff;
      border-top-left-radius: 4px;
      border-bottom-right-radius: 12px;
    }
  }

  .sourceName {
    width: 200px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    text-align: left;
    font-weight: bolder;
    font-size: 16px;
    color: #333;
    line-height: 28px;
    padding: 15px;
  }

  .sourceKey {
    width: 200px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    text-align: center;
    // font-weight: 600;
    font-size: 14px;
    height: 28px;
    line-height: 28px;
    color: #ffffff;
    position: absolute;
    top: 28px;
    background-color: rgba(119, 119, 119, 0.7);
  }

  &:hover {
    cursor: pointer;
    // box-shadow: 0 10px 30px 0 hsla(0,0%,84.3%,.5);
    transform: translate3d(0, -4px, 0);
    box-shadow: 0px 0px 8px #8D8E8F;
  }

  .sourceImg {
    width: 100%;
    height: 200px;
    overflow: hidden;
    background-position: center center;
    position: relative;

    img {
    }

    .bg {
      filter: blur(10px);
      position: absolute;
      width: 100%;
      z-index: 1;
      left: 0;
    }

    .logo {
      position: relative;
      width: 100%;
      height: 100%;
      z-index: 2;
    }
  }

  .erCode {
    .qrcode {
      width: 80%;
      margin: auto;
      position: absolute;
      display: none;
      left: 0;
      right: 0;
      top: 34px;
      // background-color: rgba(4, 1, 1, 0.53);
      // padding: 10px;
      z-index: 3;
    }
  }

  .qr {
    float: left;
    cursor: pointer;
    top: 8px;
    margin: 0 20px;
    border: 0;

    .qrBox {
      position: relative;
      top: 6px;
      background-color: #eee;
      border-radius: 3px;
      text-align: center;
      width: 36px;
      height: 36px;
    }

    &:hover {
      .qrBox {
        background-color: #1593ff;
      }

      .qrcode {
        display: block;
      }
    }

    .eqf-QRcode {
      position: absolute;
      width: 20px;
      height: 20px;
      font-size: 20px;
      background: url('../../assets/image/project/qr.png');
      background-repeat: no-repeat;
      background-size: 100% 100%;
      top: 50%;
      transform: translate(-50%, -50%);
    }
  }

  .sourceButton {
    float: left;
    height: 46px;
    bottom: 0;
    padding-right: 10px;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;

    .editButton {
      text-decoration: none;
      color: #333;
      margin: 10px 2px;
      padding: 6px 10px;
      font-size: 13px;
      cursor: pointer;
      border: 1px solid #ccd5db;
      border-radius: 4px;
      box-sizing: border-box;
      transition: background-color 0.4s ease-in-out;

      &:hover {
        border-color: #1593ff;
        color: #1593ff;
      }
    }

    .act_btn:hover {
      .actions {
        display: block;
      }
    }

    .actions {
      display: none;
      position: absolute;
      right: 21px;
      bottom: 50px;
      width: 50%;
      text-align: left;
      background-color: #fff;
      box-shadow: 0px 0px 8px #8D8E8F;
      z-index: 999;

      .action {
        font-family: PingFang-SC-Medium;
        text-decoration: none;
        color: #333;
        font-size: 13px;
        cursor: pointer;
        transition: background-color 0.4s ease-in-out;
        margin: 0;
        line-height: 25px;
        padding: 4px 10px;

        &:hover {
          background-color: #1593ff;
        }
      }

      .last {
        width: 100%;
        height: 10px;
        margin: 0;
        padding: 0;
        background-color: rgba(220, 196, 196, 0.2);
      }

      .dele:hover {
        background-color: #f56c6c;
      }
    }
  }
}
</style>