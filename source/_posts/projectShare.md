---
layout: post
title: "React+Dva+Umi+Antd项目日常开发问题清单"
date: 2021-06-14 22:37
comments: true
tags: 
	 - react
	 - typeScript
   - GGEditor
---

### 前言
  > *两个月前接触到了一个新的项目，整个项目前端工程使用Umi脚手架搭建，工程目录是由不同的模块组成，而这里的模块也是一个子工程，自己接触到的也是一个子工程，技术栈主要有 `React + Dva + Umi + Antd + GGEditor + TypeScript`，模块功能主要是做图谱可视化展示，并在此基础上做节点配置、删除，查询操作，本文着重记录项目开发过程中踩坑点。顺带过一下关键技术栈的使用。*

<!--more-->

### 技术栈简介
 - **[Dva](https://dvajs.com/)**
    > *dva 首先是一个基于 redux 和 redux-saga 的数据流方案，然后为了简化开发体验，dva 还额外内置了 react-router 和 fetch，所以也可以理解为一个轻量级的应用框架。*
    
    **使用**
    ```tsx
      // 组件里使用Dva暴露的connect方法将组件与对应model进行关联
      const AddSceneTypeModel:FC<IProps> = ({
        sceneConfig
      }):JSX.Element => {

        const confirmHandle = () => {
          validateFields((errors, values) => {
            if(errors) return;
            const {
              sceneTypeName: dictName,
            } = values;
            const dictId = require('react-native-uuid').v4().replace(/-/g, "");
            dispatch({
              type: 'sceneConfig/fetchAddSceneType',
              payload: {
                groupId: "KG_SCEGROUP",
                dictName,
                dictId,
              },
              callback: ({header: { code }}) => {
                if(code === '1') {
                  message.success('新增场景类型成功！！');
                  setDefaultVal(dictId);
                  fetchSceneTypeDict();
                }
              }
            })
            setVisible(false);
          })}

        return (
          <AntdDragModal
            visible={visible}
            zIndex={1002}
            title="新增场景分类"
            width={500}
            onCancel={() => setVisible(false)}
            onOk={confirmHandle}
          >
            <Form {...formItemLayout}>
              <FormItem label="场景分类名称">
                { 
                    getFieldDecorator('sceneTypeName', {
                      rules: [{ required: true, message: '请输入场景分类名称' }],
                    })( <Input placeholder="请输入" autoFocus /> ) 
                  }
              </FormItem>
            </Form>
          </AntdDragModal>
        )
      }

      export default connect(({ sceneConfig }) => ({
        sceneConfig
      }))(Form.create()(AddSceneTypeModel));
    ```
    ```ts
      // model定义
      export default {
        namesapce: 'sceneConfig',
        state: {
          entityData: [],
          graphInstance: {}, // 图谱实例
          graphInfo: {},
          sceneData: [],
        },

        effects: {
          /**
          * @description: 获取场景列表数据
          */
          * fetchSceneData({ payload, callback }, { call }) {
            const response = yield call(fetchSceneData, payload);
            if (callback) callback(response);
          },
        }
        reducers: {
          updateState(state, action){
            return {
              ...state,
              ...action.data
            }
          }
        },
      }
    ```
- **[Umi](https://umijs.org/zh-CN/)**
  
  > *Umi，中文可发音为乌米，是可扩展的企业级前端应用框架。Umi 以路由为基础的，同时支持配置式路由和约定式路由，保证路由的功能完备，并以此进行功能扩展。然后配以生命周期完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求。*
  
  目录结构
  ```js
  .
  ├── package.json
  ├── .umirc.ts
  ├── .env
  ├── dist
  ├── mock
  ├── public
  └── src
     ├── .umi
     ├── layouts/index.tsx
     ├── pages
         ├── index.less
         └── index.tsx
     └── app.ts
```
  
- **[GGEditor](https://www.yuque.com/blueju/gg-editor/)**
  
  > *GG-Editor 是基于 G6-Editor 进行二次封装的一款可视化操作应用框架，不过目前相关的使用说明文档太稀少了，没有一个完整的项目应用案例，还有就是核心代码 gg-editor-core 没有开源，那么今天就教大家如何基于 GG-Editor 开发一款脑图应用。*
  
   **使用**
   ```
    npm install --save gg-editor
   ```
   ```tsx
  import React, { PureComponent, createRef } from 'react';
  import { Flow, GGEditorEvent, withPropsAPI } from 'gg-editor';
  import { connect } from 'dva'
  
  @connect(({ sceneConfig }) => ({
    sceneConfig
}))
  
  class SceneCfgFlow extends PureComponent {
    state = {
      entityModal: false,
      relationModal: false,
      isNewEdge: false,
      isShowMenu: false,
      currentNodeId: '',
      currentEdgeId: '',
      menuType: '',
      shapeId: ''
    }
    render(){
      const { sceneConfig: { graphInfo } } = this.props;
      return (
        <GGEditor>
          <Flow
            className={styles.graph}
            noEndEdge={false}
            data={graphInfo}
            ref={this.flowRef}
            onAfterChange={this.onAfterChange}
          />
        </GGEditor>
      )
    }
  }
  export default withPropsAPI(SceneCfgFlow)
   ```
### 踩坑记录
  1. **组件关联model**
      ```ts
        export default connect(({ sceneConfig }) => ({
          sceneConfig
        }))(Form.create()(AddSceneTypeModel));

        // model
        export default {
          namesapce: 'sceneConfig',
          ...
        }
      ```

      > **注意：** *connect·方法里的model名称必须要与对应model文件里的namespace对应定义的名称一致而不是该文件的文件名称，否则对应组件上的props里不会存在对应model信息（组件通过connect关联model后对应model信息会绑定在组件的props属性上）*
  2. **Antd3.X版本组件使用**
      - **Form.create()包装的组件**
        ```tsx
        const SetEntity:FC<IProps> = ({
          visible = false, setVisible, nodeId,
          form: { getFieldDecorator, validateFields, setFieldsValue, getFieldsValue },
          sceneConfig: { graphInstance: graph },
          ...rest
        }):JSX.Element => {
          ....
        }

        export default connect(({ sceneConfig }) => ({
          sceneConfig,
        }))(Form.create()(SetEntity));
        ```
        > **注意：** *在组件里涉及到form表单的，是用来Form.create包装的组件里表单元素需要使用getFieldDecorator进行表单的双向绑定，这样值变化不需要手动绑定onChange事件，需要手动设置输入控件的值需使用setFieldsValue，不能使用reactHookes定义的setState方法更新输入框绑定的值，这样更新的话整个组件的state的值实际是没有同步更新的。*
      
      - **Modal组件拖拽**
        Antd组件库里的Modal组件从始至终都没有拖拽功能支持，又了解到后续也不会考虑拖拽，因此需要借助第三方插件DragM来实现。
        ```tsx
          // 安装
          npm i dragm -S

          // 新建ModalDrag文件
          import React from 'react';
          import DragM from 'dragm';
          export default class ModalDrag extends React.Component {
            updateTransform = transformStr => {
              this.modalDom.style.transform = transformStr;
            };
            componentDidMount() {
              this.modalDom = document.getElementsByClassName(
                "ant-modal-wrap"  // modal的class是ant-modal-wrap
              )[0];
            }
            render() {
              const { title } = this.props;
              return (
                <DragM updateTransform={this.updateTransform}>
                  <div>{title}</div>
                </DragM>
              );
            }
          }

          // 使用
          import React from 'react';
          import ModalDrag from './ModalDrag .js';
          class Demonstration extends React.Component {
            render(){
              const title = <ModalDrag title="标题” />
              return(
                <Modal
                  title={title}
                >
                </Modal>
              )
            }
          }
        ```

        > **注意：** *这样包装使用，当遇到同时存在多个弹窗时，拖拽最外层弹窗存在问题，就是跟着鼠标移动的不是当前title所在的弹窗而实际效果是里层的弹窗移动，解决办法需要将每个弹窗组件用一个唯一标识区分不同弹窗，获取当前操作弹窗时需根据对应唯一标识去获取。当然这样后在实际项目中测试还是存在问题，因此后面放弃了这个第三方插件，最后在网上找了一个比较好的方案就是针对antd里的Modal做二次封装的组件。*

      - **Pagination**
        ```tsx
        <Pagination
          showSizeChanger
          showQuickJumper
          onShowSizeChange={paginationHandle}
          onChange={paginationHandle}
          pageSize={pageInfo.pageSize}
          defaultPageSize={10}
          showTotal={(total) => `总共${total}条`}
          defaultCurrent={1}
          total={pageInfo.total}
        />
        ```
        > **注意：** *这里使用部分受控组件的写法，defaultCurrent={1}，pageInfo.total默认值为0，默认高亮第一页存在问题，处理办法就是，pageInfo.total设置不为0，或者改为完全受控组件写法。*
    
  3. **GGEditor**
     - **节点鼠标右键菜单**
        ```tsx
        import React, { FC } from 'react'
        import { Menu, Modal, message } from 'antd'
        import { connect } from 'dva'
        import { ContextMenu, NodeMenu, EdgeMenu } from 'gg-editor'

        import { shapeTypes } from '../../../common';

        import styles from './index.less'

        const { confirm } = Modal;

        enum EntityActType{
          editEntity=1,
          deleteEntity=2,
          manageRelative=3,
          deleteRelation=4
        }

        interface IFilterAction {
          [prop: string]: Function
        }

        type IProps = {
          menuType: string,
          shapeId: string,
          fetchRelationData: ((fromThingId: string, toThingId: string, shapeId: string) => void),
          setMenuModal: ((isShow: boolean) => void)
          updateGraphInfo: ((graph) => void),
          openShapeModal: ((shapeId: string) => void)
        }

        const MenuItem = Menu.Item;

        const ContextMenuComp:FC<IProps> = ({
          menuType, shapeId, updateGraphInfo,
          openShapeModal, fetchRelationData, setMenuModal,
          sceneConfig: { graphInstance: graph }
        }):JSX.Element => {
          // ...        
          return (
            <ContextMenu className={styles.contextMenu}>
              <MenuWrap>
                <Menu
                  mode="vertical"
                  style={{
                    width: 80,
                    textAlign: 'center',
                    border: '1px solid #e8e8e8',
                  }}
                  onClick={({ key }) => {
                    console.log(graph);
                    const actionType = EntityActType[Number(key)];
                    menuActionFilter[actionType]();
                    setMenuModal(false)
                  }}
                >
                  {
                    menuType === shapeTypes.NODE ? [
                      <MenuItem key={EntityActType.deleteEntity}>删除</MenuItem>,
                      <MenuItem key={EntityActType.editEntity}>编辑</MenuItem>
                    ] : [
                      <MenuItem key={EntityActType.deleteRelation}>删除</MenuItem>,
                      <MenuItem key={EntityActType.manageRelative}>编辑</MenuItem>
                    ]
                  }
                </Menu>
              </MenuWrap>
            </ContextMenu>
          )
        }

        export default connect(({ sceneConfig }) => ({
          sceneConfig,
        }))(ContextMenuComp);
        ```
        > *当时遇到的一个问题是，在画布上节点首次右键鼠标时，对应菜单定位到了画布的左上角，而没有定位到鼠标移到对应节点位置上。当时的处理方法通过样式控制让菜单第一次不显示处理的，就是首次触发右键菜单要两次点击才展示*
      - **GGEditor获取画布实例**
        由于GGEditor没有直接暴露出画布实例方法，因此需要通过在Flow组件是手动绑定ref， 通过组件实例去获取对应下面的画布实例及更多GGEditor没有暴露的方法都可以获取到。如下：
        ```tsx
        componentDidMount() {
          if(this.flowRef.current) {
            const { graph } = this.flowRef.current;
            this.updateGraphInstance(graph);
            this.bindEvent(graph);
          }
        }

        <Flow
          className={styles.graph}
          noEndEdge={false}
          data={graphInfo}
          ref={this.flowRef}
          onAfterChange={this.onAfterChange}
        />
        ```
        > **注意：** *Flow组件必须放在GGEidtor组件里使用，同时通过graph.render(data)手动渲染数据存在问题，浏览器有报错，请谨慎使用。*
### 最后
  > *以上列出的问题算是自己在开发过程中遇到的印象比较深的，当然还有其他的一些问题，相对列举的比较容易解决就不再一一列举了，此次分享就到这里，希望能给接触到类似技术栈的开发者有所帮助，同时避免重复踩坑。水平有限希望路过的大佬们不喜勿喷，同时就本文分享的内容有不足的地方望批评指正。一直以来在掘金平台里都是扮演读者的身份，期间看过很多大佬分享的技术文章，从中也学到了很多。*