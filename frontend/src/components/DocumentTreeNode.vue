<template>
  <Row :class="rowClass" @mouseover.native="isMouseOver=true" @mouseout.native="isMouseOver=false" @click.native="onTreeNodeClick">
    <Col :span="isMouseOver?21:24">
      <Icon v-show="data.type === 'group'" :class="toggleClass" size="14" @click="onToggleStatusChange" />
      <Icon v-show="data.type === 'data'" type="md-document" class="tree-node-inner-button" />
      <span class="tree-node-inner-text">{{data.name}}</span>
    </Col>
    <Col :span="isMouseOver?2:0" align="right">
      <Tooltip content="Delete" placement="bottom-end" :delay="500">
        <Icon v-show="isMouseOver" type="md-trash" class="tree-node-inner-button" color="#ed4014" @click.stop="shownDeleteModal = true"/>
      </Tooltip>
      <span @click.stop="">
        <Dropdown v-show="isMouseOver" placement="bottom-end" @on-click="onDropdownMenuClick" >
          <a href="javascript:void(0)">
            <Icon type="ios-more" class="tree-node-inner-button"></Icon>
          </a>
          <DropdownMenu slot="list" style="min-width:60px">
            <DropdownItem align="left" name="delete">Delete</DropdownItem>
            <DropdownItem align="left" name="copy">Copy</DropdownItem>
            <DropdownItem align="left" name="paste" :disabled="hasCopyTarget">Paste</DropdownItem>
            <DropdownItem align="left" name="addGroup" :disabled="data.type === 'data'">Add group</DropdownItem>
            <DropdownItem align="left" name="addData" :disabled="data.type === 'data'">Add data</DropdownItem>
          </DropdownMenu>
        </Dropdown>
      </span>
    </Col>
    <Modal
      v-model="shownCreateModal"
      title="Create"
      ok-text="OK"
      cancel-text="Cancel"
      @on-ok="onCreate"
      @on-cancel="createName = null"
    >
      <Row>
        <Col span="3" align="right">
          <span>Parent:</span>
        </Col>
        <Col span="18" offset="1">
          <span>{{data.name}}</span>
        </Col>
      </Row>
      <Row style="padding-top:10px">
        <Col span="3" align="right">
          <span>Name:</span>
        </Col>
        <Col span="18" offset="1">
          <Input v-model="createName" size="small" />
        </Col>
      </Row>
    </Modal>
    <Modal v-model="shownDeleteModal">
      <p slot="header" style="color:#f60;text-align:center">
        <Icon type="ios-information-circle"></Icon>
        <span>Delete confirmation</span>
      </p>
      <div style="text-align:center">
        <span style="font-size:14px">Are you sure you want to delete {{data.type}} <b>{{data.name}}</b></span>
      </div>
      <div slot="footer">
        <Button type="error" size="large" long @click="onTreeNodeDelete">Delete</Button>
      </div>
    </Modal>
  </Row>
</template>

<script>
export default {
  props: ['data', 'treestore'],
  data() {
    return {
      isMouseOver: false,
      shownDeleteModal: false,
      shownCreateModal: false,
      createName: null,
      createType: null
    }
  },
  computed: {
    rowClass() {
      if (this.$store.state.dataManager.focusNodeInfo && this.data.id === this.$store.state.dataManager.focusNodeInfo.id) {
        return ['tree-node-inner-row', 'tree-node-inner-row-select']
      } else if (this.isMouseOver) {
        return ['tree-node-inner-row', 'tree-node-inner-row-foucs']
      } else {
        return ['tree-node-inner-row']
      }
    },
    toggleClass() {
      let toggleClassObj = []
      if (this.data.open) {
        toggleClassObj.push('ivu-icon ivu-icon-md-arrow-dropdown')
      } else {
        toggleClassObj.push('ivu-icon ivu-icon-md-arrow-dropright')
      }
      if (this.data.children.length) {
        toggleClassObj.push('tree-node-inner-button')
      } else {
        toggleClassObj.push('tree-node-inner-button-empty')
      }
      return toggleClassObj
    },
    hasCopyTarget() {
      return this.$store.state.dataManager.copyTarget === null
    }
  },
  methods: {
    onDropdownMenuClick(payload) {
      if (payload === 'delete') {
        this.shownDeleteModal = true
      } else if (payload === 'copy') {
        this.onTreeNodeCopy()
      } else if (payload === 'paste') {
        this.onTreeNodePaste()
      } else if (payload === 'addGroup') {
        this.createType = 'group'
        this.shownCreateModal = true
      } else if (payload === 'addData') {
        this.createType = 'data'
        this.shownCreateModal = true
      } else {}
    },
    onToggleStatusChange() {
      this.treestore.toggleOpen(this.data)
      if (this.data.open === true) {
        this.$store.commit('addGroupListOpenNode', this.data.id)
      } else {
        this.$store.commit('deleteGroupListOpenNode', this.data.id)
      }
    },
    onTreeNodeClick() {
      this.$store.commit('setFocusNodeInfo', this.data)
      if (this.data.type === 'group') {
        this.$store.dispatch('loadGroupDetail', this.data)
      } else if (this.data.type === 'data') {
        this.$store.dispatch('loadDataDetail', this.data)
      } else {}
    },
    onTreeNodeDelete() {
      if (this.data.type === 'group') {
        this.$store.dispatch('deleteGroup', this.data)
      } else if (this.data.type === 'data') {
        this.$store.dispatch('deleteData', this.data)
      } else {}
    },
    onTreeNodeCopy() {
      // this.$store.commit('setCopyTarget', this.data)
    },
    onTreeNodePaste() {
      // let copyObj = this.$store.state.dataManager.copyTarget
      // if (copyObj.type === 'group') {
      //   this.$store.dispatch('createGroup', {
      //     groupName: copyObj.name,
      //     parentId: this.data.id})
      // } else if (copyObj.type === 'data') {
      //   this.$store.dispatch('createData', {
      //     dataName: copyObj.name,
      //     parentId: this.data.id})
      // } else {}
    },
    onCreate() {
      this.$store.commit('addGroupListOpenNode', this.data.id)
      if (this.createType === 'group') {
        this.$store.dispatch('createGroup', {
        groupName: this.createName,
        parentId: this.data.id})
      } else if (this.createType === 'data') {
        this.$store.dispatch('createData', {
        dataName: this.createName,
        parentId: this.data.id})
      } else {}
    }
  }
}
</script>

<style>
.tree-node-inner-row {
  padding: 4px 0px 4px 0px;
}
.tree-node-inner-button {
  padding-left: 5px;
  cursor: pointer;
}
.tree-node-inner-button-empty {
  padding-left: 5px;
  cursor: pointer;
  color: #c5c8ce;
}
.tree-node-inner-text {
  padding-left: 3px;
  font-size: 14px;
  cursor: pointer;
  word-break: break-all;
}
.tree-node-inner-row-select {
  background-color: #ebf7ff
}
.tree-node-inner-row-foucs{
  background-color: #f8f8f9
}
.ivu-dropdown > .ivu-select-dropdown {
  margin:0px 0px
}
</style>