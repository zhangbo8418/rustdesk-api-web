<template>
  <div>
    <el-card class="list-query" shadow="hover">
      <el-form inline label-width="80px">
        <el-form-item>
          <el-button type="primary" @click="handlerQuery">{{ T('Filter') }}</el-button>
          <el-button type="danger" @click="toAdd">{{ T('Add') }}</el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <el-card class="list-body" shadow="hover">
      <el-table :data="listRes.list" v-loading="listRes.loading" border>
        <el-table-column prop="rule" :label="T('Rule')" align="center">
          <template #default="{row}">
            <div>
              {{ rules.find(r => r.value === row.rule)?.label }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="type" :label="T('Type')" align="center">
          <template #default="{row}">
            <div>
              {{ types.find(t => t.value === row.type)?.label }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="to_id" :label="T('ShareTo')" align="center">
          <template #default="{row}">
            <div v-if="row.type===TYPE_U">
              {{ users.find(u => u.id === row.to_id)?.username }}
            </div>
            <div v-else-if="row.type===TYPE_G">
              {{ groups.find(g => g.id === row.to_id)?.name }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="created_at" :label="T('CreatedAt')" align="center"/>
        <!--        <el-table-column prop="updated_at" label="更新时间" align="center"/>-->
        <el-table-column :label="T('Actions')" align="center" class-name="table-actions" width="300" fixed="right">
          <template #default="{row}">
            <el-button @click="toEdit(row)">{{ T('Edit') }}</el-button>
            <el-button type="danger" @click="del(row)">{{ T('Delete') }}</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-card class="list-page" shadow="hover">
      <el-pagination background
                     layout="prev, pager, next, sizes, jumper"
                     :page-sizes="[10,20,50,100]"
                     v-model:page-size="listQuery.page_size"
                     v-model:current-page="listQuery.page"
                     :total="listRes.total">
      </el-pagination>
    </el-card>
    <el-dialog v-model="formVisible" width="800" :title="!formData.id?T('Create') :T('Update') " :close-on-click-modal="false">
      <el-form class="dialog-form" ref="form" :model="formData" label-width="120px">
        <el-form-item :label="T('AddressBookName')">
          {{ props.collection.name }}
        </el-form-item>
        <el-form-item :label="T('Rule')" prop="rule" required>
          <el-radio-group v-model="formData.rule">
            <el-radio v-for="item in rules" :key="item.value" :value="item.value">
              {{ item.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item :label="T('Type')" prop="type" required>
          <el-radio-group v-model="formData.type">
            <el-radio v-for="item in types" :key="item.value" :value="parseInt(item.value)">
              {{ item.label }}
            </el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item :label="T('ShareTo')" prop="g_id" required>
          <!--          <el-input-number v-model="formData.to_id"></el-input-number>-->
          <div style="width: 30%">
            <el-select v-model="formData.g_id" @change="changeGId">
              <el-option
                  v-for="item in groups"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
              ></el-option>
            </el-select>
          </div>
          <div style="width: 30%;margin-left: 20px">
            <el-select v-model="formData.u_id" v-if="formData.type===TYPE_U">
              <el-option
                  v-for="item in users.filter(u => u.group_id === formData.g_id)"
                  :key="item.id"
                  :label="item.username"
                  :value="item.id"
              ></el-option>
            </el-select>
          </div>
        </el-form-item>
        <el-form-item>
          <el-button @click="formVisible = false">{{ T('Cancel') }}</el-button>
          <el-button @click="submit" type="primary">{{ T('Submit') }}</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
  </div>
</template>

<script setup>

  import { T } from '@/utils/i18n'
  import { useRepositories } from '@/views/address_book/rule'
  import { onActivated, onMounted, watch } from 'vue'

  const props = defineProps({
    collection: {
      type: Object,
      required: true,
    },
    is_my: {
      type: Number,
      default: 0,
    },
  })
  const {
    listRes,
    listQuery,
    getList,
    handlerQuery,
    del,
    formVisible,
    formData,
    toEdit,
    toAdd,
    submit,
    rules,
    types,
    groups,
    users,
    getGroupUsers,
    TYPE_G,
    TYPE_U,
    changeGId,
  } = useRepositories(props.is_my ? 'my' : 'admin')

  formData.collection_id = props.collection.id
  formData.user_id = props.collection.user_id
  listQuery.collection_id = props.collection.id

  onMounted(getGroupUsers)
  onMounted(getList)
  onActivated(getList)

  watch(() => listQuery.page, getList)

  watch(() => listQuery.page_size, handlerQuery)

</script>

<style scoped lang="scss">

</style>
