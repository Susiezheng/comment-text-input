<template>
  <div>
    <div class="pc-form-comment">
      <h3 class="title">{{ $t('cloudpivot.formComment.pc.writeComment') }}</h3>
      <div class="text-box">
        <div class="textarea-box">
          <!-- <fakeTextarea v-model="comment" :max="2000" :isShowLimit="false" /> -->
          
          <a-texts
            ref="atPane"
            v-model="comment"
            :members="members"
            :maxlength="maxlength"
            nameKey="name"
            @touchBottom="onTouchBottom"
            @selectUser="onSelectUser"
          >
            <div
              slot="embeddedItem"
              ref="atwhoEditWrap"
              tabindex="0"
              :placeholder="$t('cloudpivot.formComment.pc.member')"
              class="atwho-edit-wrap dp-font34 editWrap"
              contenteditable="true"
              @blur="onAtwhoViewBlur"
              @input="handleInput"
              @mousedown="setDurationStart"
              @mouseup="setDurationEnd"
            ></div>
          </a-texts>
        
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
  import {
    Component, Vue, Prop, Watch
  } from 'vue-property-decorator';
  
  import ATexts from '@cloudpivot/form-comment/src/components/share/vue-at/at';

  @Component({
    name: 'PcAt',
    components: {
      ATexts,
    }
  })

  export default class PcAt extends Vue {
    comment:any = '';
    members:any = [
      {
        "id":"8a28860d7908338901790842e73e0013",
        "name":"小圆妹",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d7908338901790843ad0a0017",
        "name":"小圆宝宝",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d7908a903017908af0ec200a1",
        "name":"2342",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d790c1f9a01790d1b5bbf0aa8",
        "name":"忙忙1",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d790c1f9a01790d9c10d00dbf",
        "name":"测试用户",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d7944dbc00179463567c8046c",
        "name":"忙忙2",
        "type":3,
        "imgUrl":null
      },
      {
        "id":"8a28860d7944dbc001795560aa1c19c6",
        "name":"王丹",
        "type":3,
        "imgUrl":null
      }
    ]; // 可@人员列表
    maxlength:number = 2000; // 最大字节数

    onTouchBottom(){
    }
    // 选中人员
    onSelectUser(){
      // this.members = this.defaultMembers;
    }
    /**
     * at组件失去焦点
     * */
    onAtwhoViewBlur (index?:number) {
      setTimeout(() => {
        if (typeof index === 'number') {
          (this.$refs[`atPane${index}`][0] as any).onAtwhoViewBlur();
        } else {
          (this.$refs.atPane as any).onAtwhoViewBlur();
        }
      }, 300);
    }

    /**
     * at 组件编辑时
     * */
    handleInput(index?:any){
      let atwhoEditWrap:any;
      let atPane:any;
      if (typeof index === 'number') {
        atwhoEditWrap = null; // 回复框无需@人，故通过此方式阻断逻辑执行
        atPane = null;
      } else {
        atwhoEditWrap = (this.$refs.atwhoEditWrap as any);
        atPane = (this.$refs.atPane as any);
      }


      this.handleInputBase(atwhoEditWrap, atPane);
    }

    /**
     * 去除字符串中的html标签及内容
     * */
    filterHtmlOrContainer(str:any,isbool:boolean = false) {
      const reg =  new RegExp('<[^>]+>','gi');  //过滤所有的html标签，不包括内容

      const reg2 = new RegExp('<(img|br|hr|input)[^>]*>','gi');  //只匹配img、br、hr、input标签

      const reg3 = new RegExp('<(\\S*)[^>]*>[^<]*<\\/(\\1)>','gi');  //分组匹配，过滤所有的html标签，包括内容
      if(typeof str !='string'){  //不是字符串
        return str;
      }
      var result = str;
      if(!isbool){  //先把单标签过滤了
        result = result.replace(reg2, '');
      }
      result = result.replace(reg3,'');    //先经过分组匹配，把双标签去除，如果是嵌套标签，则会先将嵌套标签内的双标签过滤掉
      if(reg3.test(result)) { //如果为true，则代表还有标签
        return this.filterHtmlOrContainer(result, true);
      }else {
        return result;
      }
    }

    /**
     * @desc 实时搜索 当最后输入的字符是 @ 的时候, 调用接口查找列表
     * @desc 先于当前列表中搜索，有则过滤，无则加载
     * @params atwhoEditWrap获取内部html
     * */
    handleInputBase(atwhoEditWrap:any, atPane:any){
      if (!atwhoEditWrap || !atPane) return ;

      const htmlString:any = atwhoEditWrap.innerHTML;

      if (!htmlString) return ;

      const strWithoutHtmlTag = this.filterHtmlOrContainer(htmlString, false);

      if (strWithoutHtmlTag.indexOf('@') <= -1) return ; // 没有输入@

      const strAfterAt:string = strWithoutHtmlTag.substr(strWithoutHtmlTag.lastIndexOf('@')).split('@')[1].trim();

      if (!strAfterAt) {
        return;
      }

      // 先在当前列表搜索, 存在即添加
      // todo: 同名但是不在当前页如何处理
      const filteredM:Array<any> = this.members.filter((m:any) => {
        return m.name.indexOf(strAfterAt) > -1;
      });

      this.members = filteredM
    }

    time:number = 0;
    setDurationStart(){
      this.time = new Date().getTime();
    }

    setDurationEnd(){
      const now:number = new Date().getTime();
      if (now - this.time <= 200) { // 代表click
        this.doFocus();
      }
    }

    doFocus(){
      this.$nextTick(() => {
        (this.$refs.atPane as any).setFocus();
      })
    }
  }
</script>

<style lang="less" scoped>
  .pc-form-comment {
    width: 100%;
    height: 300px;
    overflow: hidden;
    &  h3.title {
      font-size: 14px;
      color:rgba(0,0,0,.65);
      line-height: 22px;
      font-weight: bold;
      margin: 0 16px;
      margin-bottom: 8px;
    }
    & > .text-box {
      border: 1px solid rgba(217,217,217,1);
      padding: 5px 12px;
      background: white;
      border-radius: 4px;
      margin: 0 16px;
      & > .textarea-box {
      }
      & > .text-box-bottom {
        display: flex;
        justify-content: space-between;
        align-items: center;
        .text-box-bottom-left {
          & > .upload-area {
            display: inline-block;
            cursor: pointer;
            .ant-upload-list-item-name {
              padding-right: 13px;
              width: 175px !important;
            }
            .icon:hover {
              color: @primary-color;
            }
          }
          & > span {
            margin-left: 8px;
            font-size: 12px;
            color: rgba(0,0,0,.45)
          }
        }
        .text-box-bottom-right {
          & > span {
            margin-right: 8px;
            font-size: 12px;
            color: rgba(0,0,0,.45)
          }
        }
      }
    }
    & > .comment-panel {
      margin-top: 12px;
      height: 100%;
      & > h3.title {
        border-bottom: 1px solid rgba(216,216,216,1);
        line-height: 26px;
      }
      & > .comment {
        height: calc( 100% - 205px);
        overflow: auto;
        .no-comment {
          text-align: center;
          position: relative;
          top: 20vh;
          font-size: 14px;
          color: rgba(1,1,1,.45);
        }
        .comment-item {
          display: flex;
          margin-bottom: 12px;
          margin: 0 16px;
          margin-bottom: 16px;
          &-left {
            margin-right: 8px;
            .avatar {
              width: 32px;
              height: 32px;
              & > img {
                width: 100%;
                height: 100%;
                border-radius: 50%;
              }
              & > i.default-avatar{
                font-size: 32px;
                line-height: 32px;
                color: #36CFC9;
              }
            }
          }
          &-right {
            flex-grow: 1;
            .username {
              font-size: 12px;
              font-weight: bold;
              color: rgba(0,0,0,0.85);
              .fr {
                float: right;
                color: rgba(0,0,0,0.65);
                font-size: 14px;
                font-weight: 400;
                cursor: pointer;
                &:hover {
                  color: @primary-color;
                }
              }
              & > .replay-txt {
                color: rgba(0, 0, 0, .45);
                font-size: 12px;
                margin-left: 8px;
                font-weight: normal;
              }
            }
            .rich-text {
              color: rgba(0,0,0,0.85);
              font-size: 14px;
              position: relative;
              word-break: break-all;
              .text-box {
                & > span[data-at-embedded] {
                  color: #107fff;
                }
              }
              .collspan {
                text-align: right;
                position: absolute;
                bottom: 2px;
                right: 0;
                width: 25%;
                background: linear-gradient(to right, rgba(255, 255, 255, 0), #f4f6fc  45%);
                color: #2970FF;
                user-select: none;
                cursor: pointer;
                &.static {
                  position: relative;
                  bottom: unset;
                  left: 2px;
                }
              }
            }
            .date-box {
              &.has-replay {
                width: 252px;
              }
              .date {
                font-size: 12px;
                color: rgba(1, 1, 1, .45);
                margin-right: 8px;
              }
              .replay-btn {
                cursor: pointer;
                user-select: none;
                font-size: 12px;
                color: #2970FF;
              }
            }
          }
        }
      }
    }
    .replay {
      margin-top: 16px;
      .comment-item {
        margin-bottom: 12px !important;
        margin-right: 0 !important;
      }
      .more-reply{
        text-align: right;
        font-size: 14px;
        color: @primary-color;
        cursor: pointer;
      }
      .reply-loaded{
        text-align: right;
        font-size: 14px;
        color: rgba(0,0,0,0.45);
      }
    }
    .fade {
      max-height: 65px;
      overflow: hidden;
    }
    .replay-box {
      margin-top: 8px;
      border: 1px solid rgba(217,217,217,1);
      background: white;
      padding: 6px;
      border-radius: 4px;
      .buttons {
        margin-top: 8px;
        text-align: right;
        & > .cancel {
          margin-right: 8px;
        }
      }
      & > .len-limit {
        text-align: right;
        font-size: 14px;
        color: rgba(0, 0, 0, .25);
      }
    }
    
    .attachment-box {
      margin-bottom: 8px;
      li.file-preview {
        width: 48px;
        height: 48px;
        position: relative;
        border:1px solid rgba(217,217,217,1);
        border-radius: 3px;
        display: inline-block;
        margin-bottom: 8px;
        margin-right: 8px;
        vertical-align: top;
        img {
          width: 100%;
          height: 100%;
        }
        .close {
          font-size: 16px;
          position: absolute;
          top: -8px;
          right: -8px;
          color: #F4454E;
          cursor: pointer;
        }
        & > i.at-type {
          display: block;
          width: 100%;
          text-align: center;
          line-height: 48px;
          font-size: 32px;
        }
        &.file-preview-icon {
          border: none;
          & > i.at-type {
            font-size: 48px!important;
          }
        }
        & > .op-mask {
          display: none;
          position: absolute;
          top: 0;
          left: 0;
          background: rgba(0, 0, 0, 0.45);
          width: 100%;
          height: 100%;
          border-radius: 3px;
          & > .op-actions {
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
            & > i {
              font-size: 12px;
              color: white;
              cursor: pointer;
              user-select: none;
              &.preview-icon {
                margin-right: 8px;
              }
            }
          }
        }
        
        &:hover {
          & > .op-mask {
            display: block;
          }
        }
      }
      &.preview {
        margin-top: 8px;
        margin-bottom: 0;
      }
    }
    /deep/.ant-upload-list {
      display: none;
    }
    .atwho-edit-wrap {
      height: 75px;
      overflow-y: auto;
      overflow-x: hidden;
      outline: none;
      color: rgba(0,0,0,0.85);
      -webkit-user-modify: read-write-plaintext-only;
      word-break: break-all;
      /deep/ span {
        color: #107fff;
      }
      &:empty:before{content: attr(placeholder);color:rgba(0, 0, 0, .25);}
      &:focus{content:none;}
    }
  }

</style>
