<template>
  <i v-if ="type === 'upload'" class="el-icon-upload" @click="deploy"/>
  <i v-else class="el-icon-download" @click="downLoad"/>
</template>

<script>
  import Utils from '@/service/Utils'
  export default {
    name: 'Deploy',
    props: {
      type: {
        type: String,
        default: 'upload'
      }
    },
    data () {
      return {
        git: {}
      }
    },
    mounted () {
      let workingDir = this.$store.state.Config.config.path
      this.git = require('simple-git/promise')(workingDir)
      window.git = this.git
    },
    methods: {
      async deploy () {
        let simpleStatus = await this.simpleStatus()
        if (simpleStatus.modified) {
          this.commit(simpleStatus.branch, 'Commit at ' + Utils.formatDate(new Date()))
        } else {
          this.$message.warning('资料库无变更')
        }
      },
      async downLoad () {
        let loading = this.$loading({
          lock: true,
          text: '与云端同步中...',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        })
        await this.git.pull()
        this.$message('同步成功')
        loading.close()
      },
      async simpleStatus () {
        let status = {modified: false, branch: 'master'}
        try {
          let statusSummary = await this.git.status()
          // console.log('statusSummary: ', statusSummary)
          if (statusSummary.modified.length > 0 || statusSummary.not_added.length > 0 || statusSummary.deleted.length > 0) {
            status.modified = true
          }
          status.branch = statusSummary.current
          return status
        } catch (e) {
          return status
        }
      },

      async commit (branch, msg) {
        let loading = this.$loading({
          lock: true,
          text: '发布中...',
          spinner: 'el-icon-loading',
          background: 'rgba(0, 0, 0, 0.7)'
        })
        try {
          await this.git.add('./*')
          await this.git.commit(msg)
          await this.git.push('origin', branch)
          this.$message('发布成功')
        } catch (e) {
          this.$message.error('发布失败')
        } finally {
          loading.close()
        }
      }

    }
  }
</script>

<style scoped>

</style>
