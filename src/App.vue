<template>
  <div id="app">
    <div class="top">
      <div class="topleft">
        <van-button type="info" :plain="type != 0" @click="type = 0" size="small">全部</van-button>
        <van-button type="info" :plain="type != 1" @click="type = 1" size="small">未开始</van-button>
        <van-button type="info" :plain="type != 2" @click="type = 2" size="small">进行中</van-button>
        <van-button type="info" :plain="type != 3" @click="type = 3" size="small"
          style="border-right: 1px solid #1989fa">已完成
        </van-button>
      </div>
      <van-button type="info" size="small" @click="isDate = true">
        <span>{{ date }}</span>
        <van-icon name="arrow-down" />
      </van-button>
      <van-calendar v-model="isDate" :min-date="minDate" :show-confirm="false" @confirm="toDate" />
    </div>
    <div class="mid">
      <div v-for="(item, index) in list" :key="index">
        <van-cell :title="
          index +
          1 +
          '. [' +
          (item.type == 3 ? '已完成' : item.type == 2 ? '进行中' : '未开始') +
          '] ' +
          item.name + item.priority
        " v-show="item.type == type || type == 0" :title-style="
  item.priority == 1
    ? 'color:red'
    : item.priority == 2
      ? 'color:orange'
      : item.priority == 3
        ? 'color:blue'
        : 'color:green'
" @click="toUpdate(item)">
          <van-icon name="edit" size="20" />
        </van-cell>
      </div>

      <van-empty description="暂无待办事项" v-show="list.length == 0" />
    </div>
    <van-icon name="add" id="add" size="50" color="#1989fa" @click="isAdd = true" />
    <van-popup v-model="isAdd" round closeable position="bottom" class="addBox">
      <p class="atitle">添加待办事项</p>
      <van-form @submit="toAdd">
        <van-field v-model="name" required autofocus label="待办事项" label-width="4.3em" placeholder="请输入待办事项" />
        <van-field name="radio" label="优先级" required label-width="4.3em">
          <template #input>
            <van-radio-group v-model="priority" direction="horizontal">
              <van-radio name="1">重要且紧急</van-radio>
              <van-radio name="2">紧急不重要</van-radio>
              <van-radio name="3" style="margin-top: 5px">重要不紧急</van-radio>
              <van-radio name="4" style="margin-top: 5px">不紧急不重要</van-radio>
            </van-radio-group>
          </template>
        </van-field>
        <van-field v-model="message" rows="2" autosize label="备注" type="textarea" label-width="4.3em"
          placeholder="请输入备注" />
        <div style="margin: 10px 16px 16px">
          <van-button round block type="info" native-type="submit">提交</van-button>
        </div>
      </van-form>
    </van-popup>
    <van-popup v-model="isUpdate" round closeable position="bottom" style="height: 342px" class="addBox">
      <p class="atitle">编辑待办事项</p>
      <van-form @submit="sendUpdate">
        <van-field v-model="upObj.name" required label="待办事项" label-width="4.3em" placeholder="请输入待办事项" />
        <van-field name="radio" label="优先级" required label-width="4.3em">
          <template #input>
            <van-radio-group v-model="upObj.priority" direction="horizontal">
              <van-radio name="1">重要且紧急</van-radio>
              <van-radio name="2">紧急不重要</van-radio>
              <van-radio name="3" style="margin-top: 5px">重要不紧急</van-radio>
              <van-radio name="4" style="margin-top: 5px">不紧急不重要</van-radio>
            </van-radio-group>
          </template>
        </van-field>
        <van-field name="radio" label="状态" required label-width="4.3em">
          <template #input>
            <van-radio-group v-model="upObj.type" direction="horizontal">
              <van-radio name="1">未开始</van-radio>
              <van-radio name="2">进行中</van-radio>
              <van-radio name="3">已完成</van-radio>
            </van-radio-group>
          </template>
        </van-field>
        <van-field v-model="upObj.message" rows="2" autosize label="备注" type="textarea" label-width="4.3em"
          placeholder="请输入备注" />
        <div style="margin: 10px 16px 16px">
          <van-button round block type="info" native-type="submit">提交</van-button>
        </div>
      </van-form>
    </van-popup>
  </div>
</template>

<script>
import { Toast } from "vant";
export default {
  name: "App",
  data() {
    return {
      type: 0,
      date: "0000/0/0",
      isDate: false,
      minDate: new Date(2022, 0, 1),
      list: [],
      allList: [],
      isAdd: false,
      name: "",
      priority: "1",
      message: "",
      isUpdate: false,
      upObj: {},
    };
  },
  watch: {
    date(e) {
      if (this.allList) {
        this.list = [];
        for (let i = 0; i < this.allList.length; i++) {
          if (this.allList[i].date == e) {
            this.list.push(this.allList[i]);
          }
        }
      }
    },
  },
  created() {
    this.allList = JSON.parse(localStorage.getItem("allList")) || [];
    this.date = this.formatDate(new Date());
  },
  mounted() {
    let loading = document.getElementById("zvie_base_loading");
    loading.style.display = "none";
  },
  methods: {
    formatDate(date) {
      return `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`;
    },
    toDate(e) {
      this.date = this.formatDate(e);
      this.isDate = false;
    },
    toAdd() {
      if (this.name == "") {
        return Toast.fail({
          position: "bottom",
          message: "待办事项不能为空",
        });
      }
      let obj = {
        id: this.allList.length + 1,
        name: this.name,
        priority: this.priority,
        type: "1",
        date: this.date,
        message: this.message,
      };
      this.list.push(obj);
      this.isAdd = false;
      Toast.success({
        position: "bottom",
        message: "提交成功",
      });
      this.allList.push(obj);
      localStorage.setItem("allList", JSON.stringify(this.allList));
      this.name = "";
      this.priority = "1";
      this.message = "";
    },
    toUpdate(e) {
      console.log(e);
      this.upObj = JSON.parse(JSON.stringify(e));
      this.isUpdate = true;
    },
    sendUpdate() {
      if (this.upObj.name == "") {
        return Toast.fail({
          position: "bottom",
          message: "待办事项不能为空",
        });
      }
      for (let i = 0; i < this.list.length; i++) {
        if (this.list[i].id == this.upObj.id) {
          this.list[i] = this.upObj;
        }
      }
      this.isUpdate = false;
      Toast.success({
        position: "bottom",
        message: "提交成功",
      });
      for (let i = 0; i < this.allList.length; i++) {
        if (this.allList[i].id == this.upObj.id) {
          this.allList[i] = this.upObj;
        }
      }
      localStorage.setItem("allList", JSON.stringify(this.allList));
      this.upObj = {};
    },
  },
};
</script>
<style scoped>
.top {
  display: flex;
  justify-content: space-between;
  margin: 10px 8px;
}

.top .van-button--small {
  height: 26px;
}

.topleft .van-button--small {
  border-right: 0;
}

#add {
  position: fixed;
  bottom: 20px;
  left: 165px;
  cursor: pointer;
}

.mid {
  width: 100%;
  height: 592px;
  overflow-y: auto;
}

.mid .van-cell {
  padding: 5px 9px;
}

.mid .van-cell__title {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 335px;
}

.mid .van-cell__value {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.addBox {
  height: 300px;
}

.addBox .atitle {
  line-height: 50px;
  color: #555;
  padding-left: 15px;
}
</style>
