<template>
  <div class="board_content">
    <b-navbar toggleable="md" type="dark" style="background-color:rgb(14,116,175);">
      <b-navbar-brand href="#" v-b-modal.changeBoardNameModal>{{boardName}}</b-navbar-brand>
      <b-nav is-nav-bar>
        <b-button-toolbar>
          <b-button-group class="mx-1" size="sm">
            <b-dropdown class="nav-button-drown" variant="default" size="sm" ref="dateDrown">
              <template slot="button-content">
                {{startDate | formatDate}}至{{endDate | formatDate}}
              </template>
              <b-card style="width:250px;border:none" no-body>
                <b-card-header style="font-size:14px;padding:10px,15px">
                  设置看板周期
                </b-card-header>
                <b-card-body>
                  <span class="time-zone">时区 GMT +8:00</span>
                  <el-date-picker type="date" placeholder="开始日期" v-model="t_startDate" size="small" :picker-options="picker_options"></el-date-picker>
                  <el-date-picker type="date" placeholder="结束日期" v-model="t_endDate" size="small" style="margin-top:10px" :picker-options="picker_options"></el-date-picker>
                </b-card-body>
                <b-card-footer>
                  <b-btn variant="success" size="sm" style="cursor:pointer" @click="saveDate">保存</b-btn>
                  <b-btn variant="default" size="sm" style="cursor:pointer" @click="cancelDate">取消</b-btn>
                </b-card-footer>
              </b-card>
            </b-dropdown>
          </b-button-group>
          <b-button-group class="mx-1" size="sm" style="height:32px">
            <b-btn variant="default" class="nav-button" v-b-modal.chartsModal>
              <icon name="area-chart"></icon>
            </b-btn>
            <b-btn variant="default" class="nav-button">
              <icon name="table"></icon>
            </b-btn>
            <b-btn variant="default" class="nav-button">
              <icon name="group" style="padding-top:2px"></icon>项目成员可见</b-btn>
            <b-btn variant="default" class="nav-button" v-b-modal.newListModal>
              <icon name="plus" style="padding-top:2px"></icon>新建列表
            </b-btn>
            <b-modal id="newListModal" size="sm" title="新建列表" @ok="newList" @shown="nListName=''" button-size="sm">
              <template slot="modal-ok">保存</template>
              <template slot="modal-cancel">取消</template>
              <b-form-input placeholder="请输入列表名" v-model="nListName"></b-form-input>
            </b-modal>
          </b-button-group>
        </b-button-toolbar>
      </b-nav>

    </b-navbar>
    <div class="board_content_main" style="margin-left:20px;font-size:14px">
      <draggable v-model="List" :options="{'ghostClass':'ghost','animation':0,'group':'dragList','handle':'.list-title'}" style="align-items: flex-start" class="card-deck" @end="dragEnd">
        <b-card style="max-width:260px;background-color:#eeeeee;margin-left:1px" no-body v-for="(item,index) in List" :key="index" class="ml-1">
          <b-card-header class="list-title" v-b-modal.changeListNameModal style="cursor:pointer" @click="selectListIndex=index">
            {{item.listName}}
            <b-badge variant="success">{{item.cardList.length}}</b-badge>
            <b-dd size="sm" variant="default" class="float-right list-title-menu" style="background-color:#e7e7e7">
              <template slot="button-content">
                <icon name="bars"></icon>
              </template>
              <b-dropdown-item-button @click.stop="delList(index)">删除列表</b-dropdown-item-button>
            </b-dd>
          </b-card-header>
          <div class="card-text" style="background-color:#eeeeee;">
            <draggable v-model="item.cardList" :options="{'ghostClass':'ghost','animation':0,'group':'description'}" :move="onMove" @update="datadragEnd" style="min-height:30px;max-height:300px;overflow:auto" @end="dragEnd">
              <div class="task_view btn btn-default" style="min-height:30px" v-for="(card,cindex) in item.cardList" :key="cindex" @mouseover.self="showEditAndDelete($event)" @mouseout.self="hideEditAndDelete($event)" v-b-modal.cardModal>
                <div class="edit-and-delete-card" style="display:block" @mouseover.self="showEditDiv">
                  <span style="cursor:pointer" @click.stop="delCard(index,cindex)">
                    <icon name="trash"></icon>
                  </span>
                </div>
                <div class="task-name-content" style="background-color:#EEEEEE">
                  <span class="card-name pull-left">{{card.cardName}}</span>
                </div>
              </div>
            </draggable>
          </div>
          <b-card-footer class="list-footer" style="border:none;height:auto">
            <b-btn size="sm" variant="link" class="add-card-button float-left" style="display:block" @click="addCard($event,index)">添加卡片</b-btn>
            <div style="display:none">
              <b-form-textarea :rows="2" :max-rows="2" style="margin:3px auto;width:98%" v-model="nCardName"></b-form-textarea>
              <b-btn class="float-left" size="sm" variant="success" @click="saveCard($event,index)" style="cursor:pointer">保存</b-btn>
              <b-btn class="float-left ml-1" size="sm" variant="default" @click="cancelSave($event)" style="cursor:pointer">取消</b-btn>
            </div>
          </b-card-footer>
        </b-card>
      </draggable>
      <div>
        <ul>
          <li v-for="(item,index) in List" :key="index">
            {{item.cardList.length}}
          </li>
        </ul>
      </div>
    </div>
    <b-modal id="changeListNameModal" size="sm" title="修改列表名" @ok="changeListName" @shown="nListName=''" button-size="sm">
      <template slot="modal-ok">保存</template>
      <template slot="modal-cancel">取消</template>
      <b-form-input placeholder="请输入列表名" v-model="nListName"></b-form-input>
    </b-modal>
    <b-modal id="cardModal" size="lg" :hide-footer="true" style="margin-top:50px">
      <div class="container" style="width:100%">
        <div class="row">
          <div class="col-md-9">
            <b-form-textarea no-resize="true" :rows="1" v-model="cardName"></b-form-textarea>
            <div class="cardDescription">
              <span>添加描述</span>
            </div>
          </div>
          <div class="col-md-3">
            <ul class="cardModal-right-ul">
              <li>测试</li>
              <li>
                <b-btn v-popover:foo.right variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="user"></icon>成员</b-btn>
                <popover name="foo">
                  <ul>
                    <li>This is a Test</li>
                    <li>This is a Test</li>
                    <li>This is a Test</li>
                  </ul>
                </popover>
              </li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="tag"></icon>标签</b-btn>
              </li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="th"></icon>工作量</b-btn>
              </li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="clock-o"></icon>截止时间</b-btn>
              </li>
              <li>操作</li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="copy"></icon>复制</b-btn>
              </li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="long-arrow-right"></icon>移动</b-btn>
              </li>
              <li>
                <b-btn variant="outline-secondary" size="sm" class="cardli-button">
                  <icon name="trash"></icon>删除</b-btn>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </b-modal>
    <b-modal id="chartsModal" size="lg" @shown="chartsShow" ok-title="保存" :ok-only="true">
      <div id="chartsContent" style="width:600px;height:400px;">

      </div>
    </b-modal>
     <b-modal id="changeBoardNameModal" size="sm" title="修改看板" @ok="changeBoardName" @shown="nBoardName=''" button-size="sm">
      <template slot="modal-ok">保存</template>
      <template slot="modal-cancel">取消</template>
      <b-form-input placeholder="请输入看板名" v-model="nBoardName"></b-form-input>
    </b-modal>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import { formatDate } from '../../common/formatDate'
import Bus from '../../bus'
export default {
  components: { draggable },
  data() {
    return {
      boardId: this.$route.params.boardId,
      boardName: '',
      right: -300,
      addCardIsOpen: false,
      nCardName: '',
      onEdit: false,
      cardList: [],
      List: [],
      startDate: '',
      endDate: '',
      t_startDate: '',
      t_endDate: '',
      nListName: '',
      picker_options: { firstDayOfWeek: 1 },
      selectListIndex: -1,
      nBoardName:'',
      cardName:''

    }
  },
  methods: {
    changeBoardName(){
      this.$ajax.post('/Board/updateBoard',{'boardId':this.boardId,'boardName':this.nBoardName}).then(res=>{
        if(res.data.errcode==0)
        this.boardName=this.nBoardName
      })
    },
    chartsShow() {
      var myChart = this.echarts.init(document.getElementById('chartsContent'));
      var columns = [];
      var data=[];
      for (let i = 0; i < this.List.length; i++)
        columns.push(this.List[i].listName);
      for(let i=0;i<this.List.length;i++)
      data.push({'value':this.List[i].cardList.length,'name':this.List[i].listName})
      var option = {
        title: {
          text: '卡片分布图'
        },
        tooltip: {},
        legend: {
          bottom: 10,
          left: 'center',
          data: columns
        },
       series:[{type:'pie',radius:'65%',center:['50%','50%'], selectedMode: 'single',data:data}]
      };
      myChart.setOption(option)
    },
    changeListName() {
      const listId = this.List[this.selectListIndex].listId;
      this.List[this.selectListIndex].listName = this.nListName;
      this.$ajax.post('/List/updateList', { 'listId': listId, 'listName': this.nListName }).then(res => {
        this.List[this.selectListIndex].listName = this.nListName;
      })
    },
    newList() {
      var params = new URLSearchParams();
      params.append('listName', this.nListName)
      if (this.List === null||this.List.length===undefined)
        params.append('listLocate', 0)
      else params.append('listLocate', this.List.length)
      params.append('boardId', this.boardId)
      this.$ajax.post('/List/newList', params, {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded"
        }
      }).then(res => this.List = res.data.data).catch(res => console.log(res))
    },
    saveDate() {
      const startDate = formatDate(new Date(this.t_startDate), 'yyyy-MM-dd')
      const endDate = formatDate(new Date(this.t_endDate), 'yyyy-MM-dd')
      this.$ajax.post('/Board/updateBoard', { 'boardId': this.boardId, 'boardStartDate': startDate, 'boardEndDate': endDate }).then(res => { if (res.data.errcode == 0) { this.endDate = this.t_endDate; this.startDate = this.t_startDate } })
      this.$refs.dateDrown.visible = false;
    },
    cancelDate() {
      this.t_endDate = this.endDate;
      this.t_startDate = this.startDate;
      this.$refs.dateDrown.visible = false;
    },
    delList(index) {
      var params = new URLSearchParams;
      params.append('listId', this.List[index].listId)
      params.append('boardId', this.boardId)
      this.$ajax.post('/List/delList', params, {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded"
        }
      }).then(res => {
        if (res.data.errcode === 0) {
          this.List.splice(index,1);
          for (var tindex in this.List) {
            this.List[tindex].listLocate = tindex;
            for (var ctindex in this.List[tindex].cardList) {
              this.List[tindex].cardList[ctindex].cardListId = this.List[tindex].listId
              this.List[tindex].cardList[ctindex].cardLocate = ctindex;
            }
          };
          this.$ajax.post("/Card/updateCardList", JSON.stringify(this.List), {
            headers: {
              "Content-Type": "application/json"
            }
          }).then(res => this.List = res.data.data).catch(res => console.log(res))
        } else alert("删除失败")
      })
    },
    delCard(index, cindex) {
      this.$ajax.post('/Card/delCard', this.List[index].cardList[cindex]).then(res => {
        if (res.data.errcode == 0) {
          this.List[index].cardList.splice(cindex, 1)
          for (var tindex in this.List) {
            this.List[tindex].listLocate = tindex;
            for (var ctindex in this.List[tindex].cardList) {
              this.List[tindex].cardList[ctindex].cardListId = this.List[tindex].listId
              this.List[tindex].cardList[ctindex].cardLocate = ctindex;
            }
          }
          this.$ajax.post("/Card/updateCardList", JSON.stringify(this.List), {
            headers: {
              "Content-Type": "application/json"
            }
          }).then(res => console.log(res)).catch(res => console.log(res))
        }
        else alert("删除失败")
      })


    },
    addCard(event) {
      if (!this.addCardIsOpen) {
        event.target.nextElementSibling.style.display = 'block'
        event.target.style.display = 'none'
        this.addCardIsOpen = true
      }
    },
    saveCard(event, index) {
      var params = new URLSearchParams();
      params.append('cardName', this.nCardName);
      params.append('cardLocate', this.List[index].cardList.length);
      params.append('cardListId', this.List[index].listId)
      this.$ajax.post('/Card/newCard', params, {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded"
        }
      }).then(res => {
        this.List[index].cardList.push(res.data.data)
        this.nCardName = ''
        for (var tindex in this.List) {
          this.List[tindex].listLocate = tindex;
          for (var ctindex in this.List[tindex].cardList) {
            this.List[tindex].cardList[ctindex].cardListId = this.List[tindex].listId
            this.List[tindex].cardList[ctindex].cardLocate = ctindex;
          }
        }
        this.$ajax.post("/Card/updateCardList", JSON.stringify(this.List), {
          headers: {
            "Content-Type": "application/json"
          }
        }).then(res => console.log(res)).catch(res => console.log(res))
        event.target.parentNode.style.display = 'none'
        event.target.parentNode.previousElementSibling.style.display = 'block'
        this.addCardIsOpen = false;
      })


    },
    cancelSave(event) {
      event.target.parentNode.style.display = 'none'
      event.target.parentNode.previousElementSibling.style.display = 'block'
      this.addCardIsOpen = false;
    },
    showEditAndDelete(event) {
      var target = event.target;
      if (target.style.backgroundColor != '#EEEEEE') {
        target.children[0].style.backgroundColor = '#EEEEEE';
        target.children[0].style.color = 'black'
      }
    },
    showEditDiv(event) {
      event.target.style.backgroundColor = '#EEEEEE';
      event.target.style.color = 'black'
    },
    hideEditAndDelete(event) {
      var target = event.target;
      target.children[0].style = ""

    },
    onMove({ relatedContext, draggedContext }) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return true
    },
    dragEnd(event) {
      console.log("dragEnd")
      for (var index in this.List) {
        this.List[index].listLocate = index;
        for (var cindex in this.List[index].cardList) {
          this.List[index].cardList[cindex].cardListId = this.List[index].listId
          this.List[index].cardList[cindex].cardLocate = cindex;
        }
      }
      this.$ajax.post("/Card/updateCardList", JSON.stringify(this.List), {
        headers: {
          "Content-Type": "application/json"
        }
      }).then(res => console.log(res)).catch(res => console.log(res))
    },
    datadragEnd(evt) {

    },
  },
  computed: {
    height() {
      return window.innerHeight - 150;
    },
  },
  watch: {
    boardId() {
      this.$ajax.post('/Board/getBoardById', { 'boardId': this.boardId }).then(result => {
        this.boardName = result.data.data.boardName;
      }).catch(res => { console.log(res) })
    }
  },
  created() {
    this.$ajax.post('/Card/getCardList', { 'boardId': this.boardId }).then(res => { this.List = res.data.data }).catch(res => (console.log(res)));
    HTMLElement.prototype.__defineGetter__("currentStyle", function() {
      return this.ownerDocument.defaultView.getComputedStyle(this, null);
    });
    this.$ajax.post('/Board/getBoardById', { 'boardId': this.boardId }).then(result => {
      this.boardName = result.data.data.boardName;
      this.startDate = result.data.data.boardStartDate;
      this.endDate = result.data.data.boardEndDate;
      this.t_endDate = this.endDate;
      this.t_startDate = this.startDate;
    }).catch(res => { console.log(res) }),
    Bus.$emit('login_ok')
  },
  filters: {
    formatDate(time) {
      var date = new Date(time);
      return formatDate(date, 'MM-dd');
    }
  }
}
</script>

<style>
.board_content {
  width: 100%;
  height: 100%;
  position: fixed;
}

.nav-button {
  -webkit-transition-duration: 0.4s;
  /* Safari */
  transition-duration: 0.4s;
  cursor: pointer;
  background-color: #0E74AF;
  color: white;
}

.nav-button:hover {
  background-color: rgba(255, 255, 255, 0.3);
  color: white;
}

.nav-button-drown>button:first-child {
  -webkit-transition-duration: 0.4s;
  /* Safari */
  transition-duration: 0.4s;
  background-color: #0E74AF;
  cursor: pointer;
  color: white;
}

.nav-button-drown>button:first-child:hover {
  background-color: rgba(255, 255, 255, 0.3);
  color: white;
}

.nav-button-drown>button:first-child::after {
  content: none;
}
.task_view {
  position: relative;
  background-color: white;
  margin: 3px auto;
  font-size: 14px;
  width: 96%;
  border: none;
  text-align: left;
  box-shadow: 1px 1px 1px 1px #c1c7c3;
}

.task_view:hover {
  background-color: #EEEEEE
}

.edit-and-delete-card {
  position: absolute;
  background-color: white;
  color: white;
  top: 0;
  right: 0;
  font-size: 13px;
  padding: 2px;
}


.list-title {
  line-height: 25px ! important;
  height: 25px !important;
  padding: 0 !important;
  text-align: left;
  border: none;
}

.list-title-menu button {
  background-color: #e7e7e7 ! important;
  cursor: pointer ! important;
}

.list-title-menu ::after {
  content: none ! important;
}

.task-top-area {
  display: flex;
}

.list-footer {
  height: 33px;
  line-height: 33px ! important;
  padding: 0 ! important;
  border: none ! important;
}

.card-name {
  padding-left: 5px;
  float: left;
}

.add-card-button {
  color: #b5b5b9;
  cursor: pointer;
}

.cardModal-right-ul {
  list-style: none;
  text-align: left;
}

.cardModal-right-ul li {
  margin-bottom: 5px;
}

.cardli-button {
  width: 115px;
  cursor: pointer;
  text-align: left;
  font-size: 13px;
}

.cardDescription {
  padding-left: 15px;
  text-align: left;
}

.time-zone {
  margin-bottom: 10px;
  display: block;
  margin-top: -5px;
  font-size: 12px;
  color: gray
}

.ghost {
  opacity: .5;
  background: #C8EBFB;
}
</style>
