<template>
  <div>
    <el-row type="flex" justify="center">
      <el-col :span="10">
        <el-form :model="form" :rules="rules" ref="form" label-width="80px">
          <el-form-item label="전화번호" prop="phones">
            <el-input type="textarea" :rows="10" placeholder="전화번호를 입력하세요..." v-model="form.phones"></el-input>
          </el-form-item>
          <el-form-item label="문자내용" prop="content">
            <el-input type="textarea" :rows="10" placeholder="문자내용을 입력하세요..." v-model="form.content"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" :loading="sendLoad" @click="sendSms">문자보내기</el-button>
            <el-button :loading="priceLoad" @click="countPrice">잔액조회</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data: () => ({
    sendLoad: false,
    priceLoad: false,
    form: {
      phones: '01074646521',
      content: null
    },
    rules: {
      phones: [
        { required: true, message: '핸드폰번호를 입력하세요.', trigger: 'change' }
      ],
      content: [
        { required: true, message: '문자내용을 입력하세요.', trigger: 'change' }
      ]
    }
  }),
  methods: {
    sendSms () {
      this.sendLoad = true
      this.$refs['form'].validate(async (valid) => {
        if (valid) {
          const chunkTo = (arr, n) => arr.length ? [arr.slice(0, n), ...chunkTo(arr.slice(n), n)] : []
          const recipients = this.form.phones.split(/\r*\n/).map(item => {
            const phone = item.replace(/-/gi, '')
            return phone === '' ? false : (phone.substr(0, 3) === '010' ? `+82${phone.substr(1)}`.trim() : `82${phone}`.trim())
          })
          const config = {
            headers: {
              Authorization: 'AccessKey VmBsqcAQnCClpUweXDRGf7jXB'
            }
          }
          const chunk = chunkTo(recipients, 50)
          for (let index = 0; index < chunk.length; index++) {
            const item = chunk[index]

            const params = {
              originator: '12048185398',
              recipients: item,
              body: this.form.content,
              datacoding: 'unicode'
            }
            console.log(params, config, item)
            // const response = await axios.post('https://rest.messagebird.com/messages', params, config)
            this.$notify({
              title: '성공',
              message: `문자 발송시작..`,
              type: 'success'
            })
          }

          this.$refs['form'].resetFields()
          this.sendLoad = false
        } else {
          this.sendLoad = false
          this.$notify.error({
            title: '실패',
            message: '내용을 확인해주세요.'
          })
          return false
        }
      })
    },
    async countPrice () {
      this.priceLoad = true
      const { data } = await axios.get('https://rest.messagebird.com/balance', {
        params: {
          access_key: 'VmBsqcAQnCClpUweXDRGf7jXB'
        }
      })
      this.$notify.info({
        title: '남은 잔액',
        message: `$ ${data.amount} 달러`
      })
      this.priceLoad = false
    }
  }
}
</script>
