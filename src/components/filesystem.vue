<template>
  <div>
    <!-- 主页面 -->
    <el-container>
      <el-header>
        <el-row>
          <el-col :span="2"><el-tag>2052324 方新宇</el-tag> </el-col>
          <el-col :span="2"><el-tag>文件管理系统</el-tag> </el-col>

          <el-col :span="2"
            ><el-tag>当前路径:{{ curPath }}</el-tag>
          </el-col>
          <el-col :span="2" :offset="7">
            <el-button type="primary" @click="showFormatCheckbox = true"
              >将磁盘格式化</el-button
            >
          </el-col>
          <!-- 上一级按钮 -->
          <el-col :span="2" :offset="1"
            ><el-button
              type="primary"
              icon="el-icon-back"
              @click="back"
              :disabled="this.pathList.length === 1"
            ></el-button
          ></el-col>
          <!-- 新建文件夹按钮 -->
          <el-col :span="2">
            <el-button
              type="primary"
              icon="el-icon-folder-add"
              @click="showDialogDir = true"
            ></el-button>
          </el-col>
          <!-- 新建文件按钮 -->
          <el-col :span="2">
            <el-button
              type="primary"
              icon="el-icon-document-add"
              @click="showDialogFile = true"
            ></el-button>
          </el-col>
          <!-- <el-col :span="3">
            <el-input
              v-model="input"
              placeholder="请输入要创建的文件名"
            ></el-input>
          </el-col> -->
        </el-row>
      </el-header>
      <el-container>
        <!-- 磁盘侧边栏 -->
        <el-aside>
          <el-row>磁盘块</el-row>
          <el-table :data="block" stripe style="width: 100%" height="700px">
            <el-table-column prop="blockNo" label="编号"> </el-table-column>
            <el-table-column label="存放内容" width="150">
              <template #default="scope">
                <span style="margin-left: 5px">{{
                  scope.row.blockSimCon
                }}</span>
              </template>
            </el-table-column>
            <el-table-column prop="nextBlock" label="链接块号">
            </el-table-column>
          </el-table>
        </el-aside>

        <el-container>
          <el-main>
            <el-table
              :data="currentDirectoryList"
              style="width: 100%"
              empty-text="文件夹为空"
              height="700px"
            >
              <el-table-column label="名称">
                <template slot-scope="scope">
                  <div style="display: flex; align-items: center">
                    <img
                      :src="geticonUrl(scope.row.isDirectory)"
                      class="MineIcon"
                    />
                    <span style="margin-left: 10px">{{
                      scope.row.filename
                    }}</span>
                  </div>
                  <!-- <el-tag
                    :class="
                      scope.row.isDirectory
                        ? 'el-icon-folder'
                        : 'el-icon-document'
                    "
                  >
                    {{ scope.row.filename }}
                  </el-tag> -->
                </template>
              </el-table-column>
              <el-table-column label="种类">
                <template slot-scope="scope">
                  <span>{{ scope.row.isDirectory ? "文件夹" : "文件" }}</span>
                </template>
              </el-table-column>
              <el-table-column prop="createTime" label="创建日期">
              </el-table-column>
              <el-table-column prop="latestModifiedTime" label="最新修改日期">
              </el-table-column>
              <el-table-column prop="len" label="大小(单位:KB)">
              </el-table-column>
              <el-table-column prop="startPos" label="物理地址">
              </el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button size="mini" @click="handleOpen(scope.row)"
                    >打开</el-button
                  >
                  <el-button
                    size="mini"
                    type="danger"
                    @click="checkDelete(scope.row.filename)"
                    >删除</el-button
                  >
                </template>
              </el-table-column>
            </el-table>
          </el-main>
        </el-container>
      </el-container>
    </el-container>
    <!-- 新建文件对话框 -->
    <el-dialog :visible.sync="showDialogFile" title="新建文件">
      <el-form :model="newName">
        <el-form-item label="请输入文件名称">
          <el-input v-model="newName" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showDialogFile = false">取消</el-button>
          <el-button type="primary" @click="addFileOrSubDirectory(1)"
            >确认</el-button
          >
        </span>
      </template>
    </el-dialog>
    <!-- 新建文件夹对话框 -->
    <el-dialog :visible.sync="showDialogDir" title="新建文件夹">
      <el-form :model="newName">
        <el-form-item label="请输入文件夹名称">
          <el-input v-model="newName" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showDialogDir = false">取消</el-button>
          <el-button type="primary" @click="addFileOrSubDirectory(0)"
            >确认</el-button
          >
        </span>
      </template>
    </el-dialog>
    <!-- 输入文件内容对话框 -->
    <el-dialog
      :title="openFileName"
      :visible.sync="this.showDialogContent"
      fullscreen
    >
      <el-row>
        <el-input
          type="textarea"
          fullscreen
          placeholder=""
          v-model="textContent"
        >
        </el-input>
      </el-row>
      <el-row>
        <el-col :span="1" :offset="20">
          <el-button type="text" @click="save">保存</el-button>
        </el-col>
        <el-col :span="1" :offset="1">
          <el-button type="text" @click="showDialogContent = false"
            >退出</el-button
          >
        </el-col>
      </el-row>
    </el-dialog>
    <!-- 删除文件或文件夹确认对话框 -->
    <el-dialog :visible.sync="this.showDeleteCheckbox" title="确认是否删除">
      <span>确认要删除文件或文件夹吗？此操作不可逆！</span>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showDeleteCheckbox = false">取消</el-button>
          <el-button type="primary" @click="handleDelete">确认</el-button>
        </span>
      </template>
    </el-dialog>
    <!-- 格式化确认对话框 -->
    <el-dialog :visible.sync="this.showFormatCheckbox" title="确认是否删除">
      <span>确认要格式化磁盘吗？此操作不可逆！</span>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="showFormatCheckbox = false">取消</el-button>
          <el-button type="primary" @click="initall">确认</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script>
// import { menusEvent } from "vue3-menus";
import { Notification } from "element-ui";
// import { ElMessage } from "element-plus";
export default {
  name: "Home",
  data() {
    // from website
    // 对Date的扩展，将 Date 转化为指定格式的String
    // 月(M)、日(d)、小时(h)、分(m)、秒(s)、季度(q) 可以用 1-2 个占位符，
    // 年(y)可以用 1-4 个占位符，毫秒(S)只能用 1 个占位符(是 1-3 位的数字)
    // 例子：
    // (new Date()).Format("yyyy-MM-dd hh:mm:ss.S") ==> 2006-07-02 08:09:04.423
    // (new Date()).Format("yyyy-M-d h:m:s.S")      ==> 2006-7-2 8:9:4.18
    Date.prototype.format = function (fmt) {
      var o = {
        "M+": this.getMonth() + 1, //月份
        "d+": this.getDate(), //日
        "h+": this.getHours(), //小时
        "m+": this.getMinutes(), //分
        "s+": this.getSeconds(), //秒
        "q+": Math.floor((this.getMonth() + 3) / 3), //季度
        S: this.getMilliseconds(), //毫秒
      };
      if (/(y+)/.test(fmt)) {
        fmt = fmt.replace(
          RegExp.$1,
          (this.getFullYear() + "").substr(4 - RegExp.$1.length)
        );
      }
      for (var k in o) {
        if (new RegExp("(" + k + ")").test(fmt)) {
          fmt = fmt.replace(
            RegExp.$1,
            RegExp.$1.length == 1
              ? o[k]
              : ("00" + o[k]).substr(("" + o[k]).length)
          );
        }
      }
      return fmt;
    };
    var directoryItem = function (
      filename,
      isDirectory,
      startPos,
      length = 1,
      createTime = new Date().format("yyyy-MM-dd hh:mm:ss"),
      latestModifiedTime = new Date().format("yyyy-MM-dd hh:mm:ss")
    ) {
      this.filename = filename;
      this.isDirectory = isDirectory; //是否是子目录
      this.startPos = startPos; //指向下一级目录的块号或是文件的物理地址
      this.createTime = createTime;
      this.latestModifiedTime = latestModifiedTime;
      this.len = length;
    };
    var directory = function () {
      this.dirList = new Array();
    }; //文件夹特有的数据结构——文件列表
    var block = function (blockNo, nextBlock, blockContent, blockSimCon) {
      this.blockNo = blockNo;
      this.content = blockContent;
      this.blockSimCon = blockSimCon; //简略内容
      this.nextBlock = nextBlock; //链接
    }; //磁盘块数据结构
    return {
      createItem: directoryItem, //1个块装32个item
      createDirectory: directory,
      createBlock: block,

      block: [], //物理块
      rootDirectory: Number, //根目录所在块号
      currentDirectory: Number, //当前所在目录块号
      blockNum: Number, //总共200个块
      blockSize: Number, //每个块content 32KB大
      freeBlock: Number, //首个空闲块的索引

      //路径栈 每一项均为item
      pathList: [], //路径,对相应目录项的引用

      //输入条
      input: "",
      textContent: "",
      showDialogContent: false,
      openFileName: "",
      openfileItem: "",

      //新建相关
      //新文件夹或文件名字
      newName: "",
      //对话窗是否出现
      showDialogFile: false,
      showDialogDir: false,
      //需删除的文件或文件夹名字
      deleteName: "",
      showDeleteCheckbox: false,
      showFormatCheckbox: false,
      isChanged:false,
    };
  },
  computed: {
    
    //获取当前目录下所有文件及文件夹 懒更新了 原因不知道
    currentDirectoryList: function () {
      console.log("init");
      console.log(this.isChanged);
      // this.isChanged = false;
      var curItemArray = new Array();
      var cur = this.currentDirectory; //块号
      while (cur !== -1) {
        for (var i = 0; i < this.block[cur].content.dirList.length; i++) {
          //console.log(this.block[cur].content.dirList[i]);
          curItemArray.push(this.block[cur].content.dirList[i]);
        }
        cur = this.block[cur].nextBlock;
      }
      console.log("update");
      console.log(curItemArray);
      return curItemArray;
    },
    //形成当前路径——仅供当前路径指示使用
    curPath: function () {
      var str = "";
      str += this.pathList[0].filename;
      for (var i = 1; i < this.pathList.length; i++) {
        str += "\\" + this.pathList[i].filename;
      }
      return str;
    },
  },
  created() {
    var checkInfor = JSON.parse(localStorage.getItem("rootDir"));
    console.log(checkInfor);
    this.showFormatCheckbox = false;
    this.showDeleteCheckbox = false;
    this.isChanged = false;
    this.input = "";
    this.textContent = "";
    this.showDialogContent = false;
    this.showDialogDir = false;
    this.showDialogFile = false;
    this.openfileItem = "";
    this.openFileName = "";

    this.pathList = [];
    this.block = [];

    //created
    this.blockNum = 200;
    this.blockSize = 32;
    //初始化磁盘块，存储信息
    if (checkInfor != null)
      this.block = JSON.parse(localStorage.getItem("blockInfor"));
    else {
      for (var i = 0; i < this.blockNum; i++) {
        this.block.push(new this.createBlock(i, i + 1)); //创建物理块
      }
      this.block[this.blockNum - 1].nextBlock = -1; //最后一块封闭
    }

    //空闲链表
    this.freeBlock =
      checkInfor != null
        ? JSON.parse(localStorage.getItem("freeBlockNo"))
        : 0;

    //设定根目录与当前目录
    if (checkInfor != null) {
      this.rootDirectory = JSON.parse(localStorage.getItem("rootDir"));
      this.currentDirectory = JSON.parse(localStorage.getItem("curDir"));
      this.pathList = JSON.parse(localStorage.getItem("pathList"));
    } else {
      this.rootDirectory = this.allocateFreeBlockForDirectory(); //分配块给根目录
      this.block[this.rootDirectory].blockSimCon =
        "C:" + this.block[this.rootDirectory].blockSimCon;
      this.pathList.push(new this.createItem("C:", true, 0));
      this.currentDirectory = this.rootDirectory; //当前处在根目录
    }
    console.log(this.pathList);
  },
  methods: {
    //格式化磁盘
    initall: function () {
      this.showFormatCheckbox = false;
      this.showDeleteCheckbox = false;
      this.isChanged = false;
      this.input = "";
      this.textContent = "";
      this.showDialogContent = false;
      this.showDialogDir = false;
      this.showDialogFile = false;
      this.openfileItem = "";
      this.openFileName = "";

      this.pathList = [];
      this.block = [];

      //created
      this.blockNum = 200;
      this.blockSize = 32;
      //初始化磁盘块，存储信息
      for (var i = 0; i < this.blockNum; i++) {
        this.block.push(new this.createBlock(i, i + 1)); //创建物理块
      }
      this.block[this.blockNum - 1].nextBlock = -1; //最后一块封闭

      //空闲链表
      this.freeBlock = 0;

      //设定根目录
      this.rootDirectory = this.allocateFreeBlockForDirectory(); //分配块给根目录
      this.block[this.rootDirectory].blockSimCon =
        "C:" + this.block[this.rootDirectory].blockSimCon;
      this.pathList.push(new this.createItem("C:", true, 0));
      this.currentDirectory = this.rootDirectory; //当前处在根目录
    },
    //返回上一级
    back: function () {
      this.pathList.pop(); //回退
      console.log("back");
      console.log(this.pathList);
      this.currentDirectory = this.pathList[this.pathList.length - 1].startPos; //更新回退后目录所在的区的第一个物理块号
      console.log(this.currentDirectory);
    },
    //根据起始块号获取全部文件内容 文本
    getFileContent: function (blockNo) {
      var content = "";
      while (blockNo !== -1) {
        content += this.block[blockNo].content;
        blockNo = this.block[blockNo].nextBlock; //链接到下一块
      }
      return content;
    },
    //打开文件或文件夹
    handleOpen: function (dirItem) {
      if (dirItem.isDirectory === true) {
        //打开的是文件夹
        console.log(dirItem);
        this.currentDirectory = dirItem.startPos; //将当前目录的首块号更新
        this.pathList.push(dirItem);
        console.log("pathlist");
        console.log(this.pathList);
      } else {
        //文件
        this.openFileName = dirItem.filename; //获取文件名
        this.textContent = this.getFileContent(dirItem.startPos); //获取打开文件的文本内容
        this.openfileItem = dirItem;
        this.showDialogContent = true;
      }
    },
    save: function () {
      var content = this.getFileContent(this.openfileItem.startPos); //获取该文件的块中内容
      console.log(this.openfileItem);
      if (content === this.textContent) {
        this.showDialogContent = false;
        return; //内容不变 无需保存
      } else {
        //将其块回收并进行重分配
        //回收
        var recycleBlockNo = this.openfileItem.startPos;
        this.openfileItem.startPos = -1;
        this.recycleBlock(recycleBlockNo);

        //分配，每块320个字符
        var contentList = new Array(),
          i = 0;
        while (i < this.textContent.length) {
          contentList.push(this.textContent.substr(i, this.blockSize * 10)); //将文本拆分 更新contentlist
          i += this.blockSize * 10;
        }

        var cur = -1,
          first = true;
        while (contentList.length !== 0) {
          //将contentlist内容逐元素搬运至块中
          var p = this.allocateFreeBlockForFile();
          if (p === undefined) {
            alert("无空闲空间分配");
            return;
          }
          if (first === true) {
            //对首块做特殊处理
            this.openfileItem.startPos = p; //标记文件开始位置
            cur = this.openfileItem.startPos;
            first = false;
          } else {
            this.block[cur].nextBlock = p; //链接
            cur = p;
          }
          //对当前块内容赋值
          this.block[cur].content = contentList.shift();
          this.block[cur].blockSimCon = this.block[cur].content.slice(0, 10);
        }
        //更新文件长度及最后修改日期
        this.openfileItem.len = (this.textContent.length * 1.0) / 10;
        console.log("update length");
        this.openfileItem.latestModifiedTime = new Date().format(
          "yyyy-MM-dd hh:mm:ss"
        );
        //更新该路径上的最新修改时间
        for (i = 0; i < this.pathList.length; i++) {
          this.pathList[i].latestModifiedTime = new Date().format(
            "yyyy-MM-dd hh:mm:ss"
          );
        }
        //保存完毕 关闭文件
        this.showDialogContent = false;
      }
    },
    //回收磁盘块 nowdeleBlock为要回收的磁盘块序列的起始
    recycleBlock: function (nowdeleteBlock) {
      while (nowdeleteBlock !== -1) {
        //将nowdeleteBlock为首的后续存储磁盘块释放回收
        this.block[nowdeleteBlock].blockSimCon = "";
        this.block[nowdeleteBlock].content = "";
        var p = this.block[nowdeleteBlock].nextBlock;
        this.block[nowdeleteBlock].nextBlock = this.freeBlock;
        this.freeBlock = nowdeleteBlock;
        nowdeleteBlock = p;
      }
    },
    //获取删除列表 dirStartBlockNo为目录存放区的块startPos
    getDeleteList: function (dirStartBlockNo, delArray) {
      if (dirStartBlockNo === -1) {
        return;
      }
      //对每一块的dirlist做递归操作
      while (dirStartBlockNo !== -1) {
        for (
          var i = 0;
          i < this.block[dirStartBlockNo].content.dirList.length;
          i++
        ) {
          if (
            this.block[dirStartBlockNo].content.dirList[i].isDirectory === true
          ) {
            //是子文件夹
            this.getDeleteList(
              this.block[dirStartBlockNo].content.dirList[i].startPos,
              delArray
            ); //递归删除 该子文件夹下属文件及文件夹
          }
          //将子文件夹本身放入删除列表
          delArray.push(
            this.block[dirStartBlockNo].content.dirList[i].startPos
          );
        }
        dirStartBlockNo = this.block[dirStartBlockNo].nextBlock;
      }
      console.log(delArray);
    },
    //供外界调用 删除句柄 filename为要删除的文件
    handleDelete: function () {
      this.showDeleteCheckbox = false;
      var filename = this.deleteName;
      var curDirStartBlock = this.currentDirectory,
        pre = -1;
      //查找该目录条目
      while (curDirStartBlock !== -1) {
        for (
          var i = 0;
          i < this.block[curDirStartBlock].content.dirList.length;
          i++
        ) {
          //基于当前目录 向下查找
          if (
            this.block[curDirStartBlock].content.dirList[i].filename ===
            filename
          ) {
            //是目录则递归删除
            if (
              this.block[curDirStartBlock].content.dirList[i].isDirectory ===
              true
            ) {
              var delArray = new Array();
              delArray.push(
                this.block[curDirStartBlock].content.dirList[i].startPos
              ); //将当前目录放入删除列表
              this.getDeleteList(
                this.block[curDirStartBlock].content.dirList[i].startPos,
                delArray
              ); //获取删除列表
              for (var j = 0; j < delArray.length; j++) {
                var nowdeleteBlock = delArray[j];
                this.recycleBlock(nowdeleteBlock);
              }
            }
            //是文件，直接删除即可
            else {
              var nowdeleteBlock_file =
                this.block[curDirStartBlock].content.dirList[i].startPos;
              this.recycleBlock(nowdeleteBlock_file);
            }
            //删除块中dirlist的目录条目
            this.block[curDirStartBlock].content.dirList.splice(i, 1);
            for (i = 0; i < this.pathList.length; i++) {
              this.pathList[i].latestModifiedTime = new Date().format(
                "yyyy-MM-dd hh:mm:ss"
              ); //更新修改时间
            }

            if (
              this.block[curDirStartBlock].content.dirList.length === 0 &&
              curDirStartBlock !== 0
            ) {
              //确认该块内所有子文件及文件夹已删除完毕
              console.log("Finish recycle");
              //回收该块
              if (pre !== -1) {
                this.block[pre].nextBlock =
                  this.block[curDirStartBlock].nextBlock;
              } else {
                this.pathList[this.pathList.length - 1].startPos =
                  this.block[curDirStartBlock].nextBlock;
              }
              this.block[curDirStartBlock].nextBlock = this.freeBlock;
              this.freeBlock = curDirStartBlock;
            }
            return;
          } //
        }
        pre = curDirStartBlock;
        console.log("pre" + pre);
        curDirStartBlock = this.block[curDirStartBlock].nextBlock;
      }
      this.deleteName = "";
      this.isChanged=!this.isChanged;
    },
    //检查当前目录下是否重名
    checkSameName: function () {
      for (var i = 0; i < this.currentDirectoryList.length; i++) {
        console.log("check");
        if (this.currentDirectoryList[i].filename == this.newName) {
          return true;
        }
      }
      return false;
    },
    //确认是否删除
    checkDelete: function (deleteName) {
      this.deleteName = deleteName;
      this.showDeleteCheckbox = true;
    },
    // curPath: function () {
    //   var str = "";
    //   str += this.pathList[0].filename;
    //   for (var i = 1; i < this.pathList.length; i++) {
    //     str += "\\" + this.pathList[i].filename;
    //   }
    //   return str;
    // },
    //为目录分配块空间（单块）
    allocateFreeBlockForDirectory: function () {
      if (this.freeBlock !== -1) {
        var dirBlock = this.freeBlock; //获取最前空闲块
        this.freeBlock = this.block[this.freeBlock].nextBlock; //空闲块指针后移
        this.block[dirBlock].nextBlock = -1; //将返还的块 封闭
        this.block[dirBlock].content = new this.createDirectory(); //将块初始化 磁盘块的内容为directory列表
        this.block[dirBlock].blockSimCon = "目录项"; //目录项
        return dirBlock;
      }
      alert("无空闲空间可分配");
      return undefined;
    },
    //为文件分配块空间（单块）
    allocateFreeBlockForFile: function () {
      if (this.freeBlock !== -1) {
        var free = this.freeBlock; //获取最靠前的空闲块
        this.freeBlock = this.block[this.freeBlock].nextBlock; //空闲块指针后移
        this.block[free].nextBlock = -1; //只返回单块空闲块
        this.block[free].content = ""; //初始化块内容,为文本，每个块能装32个字符
        this.block[free].blockSimCon = ""; //初始化块缩略内容,为文本，每个块能装32个字符
        return free;
      }
      alert("无空闲空间可分配");
      return undefined;
    },

    //choice 为 1 为file ;0为目录
    addFileOrSubDirectory: function (isaddFile) {
      this.newName = this.newName.trim();
      if (this.checkSameName() === true) {
        alert("同名文件或文件夹已经创建！");
        this.newName = "";
        return;
      }
      if (this.newName === "") {
        alert("名字不能为空！");
        return;
      }
      var free;
      if (isaddFile) {
        //添加文件 不取空闲块
        free = -1;
      } else {
        //取出一个空闲块分配给该子目录
        free = this.allocateFreeBlockForDirectory();
        this.block[free].blockSimCon =
          this.curPath + "\\" + this.newName + this.block[free].blockSimCon;
        if (free === undefined) {
          return;
        }
      }

      //添加目录项
      if (
        this.block[this.currentDirectory].content.dirList.length <
        this.blockSize
      ) {
        //大小还够 还可以添加在当前目录块
        //初始化为文件 不分配空间
        if (isaddFile) {
          this.block[this.currentDirectory].content.dirList.push(
            new this.createItem(this.newName, false, free, 0)
          );
        }
        //初始化为目录项 分配空间
        else {
          this.block[this.currentDirectory].content.dirList.push(
            new this.createItem(this.newName, true, free)
          );
        }
      } else {
        //向后寻找看是否有扩展目录
        var pre = this.currentDirectory,
          cur = this.block[this.currentDirectory].nextBlock;
        while (cur !== -1) {
          if (this.block[cur].content.dirList.length === this.blockSize) {
            //说明该块满了
            pre = cur;
            cur = this.block[cur].nextBlock;
          } //找到空闲块
          else {
            break;
          }
        }

        if (cur !== -1) {
          //找到已链接的空闲块
          //初始化为文件 不分配空间
          if (isaddFile) {
            this.block[cur].content.dirList.push(
              new this.createItem(this.newName, false, free, 0)
            );
          }
          //初始化为目录项 分配空间
          else {
            this.block[cur].content.dirList.push(
              new this.createItem(this.newName, true, free)
            );
          }
        } //未找到空闲块
        else {
          var nextfree = this.allocateFreeBlockForDirectory(); //再分配一块 放目录项
          this.block[nextfree].blockSimCon =
            this.curPath +
            "\\" +
            this.newName +
            this.block[nextfree].blockSimCon;
          if (nextfree === undefined) {
            return;
          }
          this.block[pre].nextBlock = nextfree; //将目录项链接
          this.block[nextfree].content.dirList.push(
            new this.createItem(this.newName, true, free)
          );
          this.pathList[this.pathList.length - 1].length++;
          console.log("addSubDirectory pathList");
          console.log(this.pathList);
        }
      }
      for (var i = 0; i < this.pathList.length; i++) {
        this.pathList[i].latestModifiedTime = new Date().format(
          "yyyy-MM-dd hh:mm:ss"
        ); //更新最新修改时间
      }
      if (isaddFile) this.showDialogFile = false;
      else this.showDialogDir = false;

      this.newName = "";
      console.log("finish");
      this.isChanged=!this.isChanged;
      console.log("init1");

      var curItemArray = new Array();
      var cur1 = this.currentDirectory; //块号
      while (cur1 !== -1) {
        for (var ii = 0; ii < this.block[cur1].content.dirList.length; ii++) {
          //console.log(this.block[cur].content.dirList[i]);
          curItemArray.push(this.block[cur1].content.dirList[ii]);
        }
        cur1 = this.block[cur1].nextBlock;
      }
      console.log("update1");
      console.log(curItemArray);
      this.currentDirectoryList = curItemArray;

      // var test = this.currentDirectoryList;
      //console.log(this.currentDirectoryList);
    },
    geticonUrl: function (isDirectory) {
      return isDirectory
        ? require("../assets/folder.jpg")
        : require("../assets/document.jpg");
    },
  },
  mounted() {
    //浏览器退出时存储
    window.onbeforeunload = (e) => {
      e = e || window.event;
      if (e) {
        e.returnValue = "关闭提示";
      }
      localStorage.clear();
      let blockInforJSON = JSON.stringify(this.block);
      let rootDirJSON = JSON.stringify(this.rootDirectory);
      let curDirJSON = JSON.stringify(this.currentDirectory);
      let pathListJSON = JSON.stringify(this.pathList);
      let freeBlockJSON = JSON.stringify(this.freeBlock);
      localStorage.setItem("blockInfor", blockInforJSON);
      localStorage.setItem("rootDir", rootDirJSON);
      localStorage.setItem("curDir", curDirJSON);
      localStorage.setItem("pathList", pathListJSON);
      localStorage.setItem("freeBlockNo", freeBlockJSON);
      return "关闭提示";
    };
    Notification({
      title: "请使用同一浏览器",
      message:
        '使用同一浏览器打开，即可自动恢复上次编辑后的资源目录。关闭浏览器时无需在意浏览器提示，选择"离开"即可',
      type: "info",
      duration: 5000,
    });
    setTimeout(
      () =>
        Notification({
          title: "说明",
          message:
            "本系统目录项占用磁盘空间,磁盘块共200块,每块32KB(1KB可存10个字符或1个目录项),初创文件时文件为空,大小为0,文件夹大小不包括子文件(夹)",
          type: "info",
          duration: 5000,
        }),
      2000
    );
  },
};
</script>

<style scoped>
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}

.MineIcon {
  width: 30px;
  height: 30px;
}

.box-card {
  width: 480px;
}
</style>