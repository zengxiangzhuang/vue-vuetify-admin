<script setup lang="ts">
import { computed, ref } from "vue";

type UserStatus = "启用" | "禁用";

interface UserItem {
  id: number;
  avatar: string;
  name: string;
  phone: string;
  status: UserStatus;
}

interface UserForm {
  avatar: string;
  name: string;
  phone: string;
  status: UserStatus;
}

const menuItems = ["Dashboard", "用户管理", "订单列表"];
const activeMenu = ref("用户管理");

const currentUserName = "管理员";

const users = ref<UserItem[]>([
  {
    id: 1,
    avatar: "https://i.pravatar.cc/100?img=11",
    name: "张三",
    phone: "13800001111",
    status: "启用",
  },
  {
    id: 2,
    avatar: "https://i.pravatar.cc/100?img=23",
    name: "李四",
    phone: "13900002222",
    status: "禁用",
  },
  {
    id: 3,
    avatar: "https://i.pravatar.cc/100?img=31",
    name: "王五",
    phone: "13700003333",
    status: "启用",
  },
  {
    id: 4,
    avatar: "https://i.pravatar.cc/100?img=17",
    name: "赵六",
    phone: "13600004444",
    status: "启用",
  },
  {
    id: 5,
    avatar: "https://i.pravatar.cc/100?img=45",
    name: "孙七",
    phone: "13500005555",
    status: "禁用",
  },
]);

const searchKeyword = ref("");
const page = ref(1);
const itemsPerPage = ref(5);

const headers = [
  { title: "头像", key: "avatar", sortable: false, align: "start" as const },
  { title: "姓名", key: "name", align: "start" as const },
  { title: "电话", key: "phone", align: "start" as const },
  { title: "状态", key: "status", align: "start" as const },
  { title: "操作", key: "actions", sortable: false, align: "start" as const },
];

const createDefaultForm = (): UserForm => ({
  avatar: "",
  name: "",
  phone: "",
  status: "启用",
});

const defaultAvatar = "https://i.pravatar.cc/100";
const userForm = ref<UserForm>(createDefaultForm());

const dialogVisible = ref(false);
const dialogMode = ref<"create" | "edit">("create");
const editingId = ref<number | null>(null);

const isFormValid = computed(() => {
  return (
    userForm.value.name.trim().length > 0 &&
    /^1\d{10}$/.test(userForm.value.phone) &&
    userForm.value.avatar.trim().length > 0
  );
});

const filteredUsers = computed(() => {
  const keyword = searchKeyword.value.trim().toLowerCase();
  if (!keyword) {
    return users.value;
  }

  return users.value.filter((user) => {
    return (
      user.name.toLowerCase().includes(keyword) ||
      user.phone.includes(keyword) ||
      user.status.includes(keyword)
    );
  });
});

const pagedUsers = computed(() => {
  const start = (page.value - 1) * itemsPerPage.value;
  return filteredUsers.value.slice(start, start + itemsPerPage.value);
});

const pageCount = computed(() => {
  return Math.max(1, Math.ceil(filteredUsers.value.length / itemsPerPage.value));
});

const dialogTitle = computed(() => {
  return dialogMode.value === "create" ? "新增用户" : "编辑用户";
});

const openCreateDialog = () => {
  dialogMode.value = "create";
  editingId.value = null;
  userForm.value = {
    ...createDefaultForm(),
    avatar: `${defaultAvatar}?u=${Date.now()}`,
  };
  dialogVisible.value = true;
};

const openEditDialog = (user: UserItem) => {
  dialogMode.value = "edit";
  editingId.value = user.id;
  userForm.value = {
    avatar: user.avatar,
    name: user.name,
    phone: user.phone,
    status: user.status,
  };
  dialogVisible.value = true;
};

const closeDialog = () => {
  dialogVisible.value = false;
};

const submitForm = () => {
  if (!isFormValid.value) {
    return;
  }

  const payload: UserItem = {
    id: editingId.value ?? Date.now(),
    avatar: userForm.value.avatar.trim(),
    name: userForm.value.name.trim(),
    phone: userForm.value.phone.trim(),
    status: userForm.value.status,
  };

  if (dialogMode.value === "create") {
    users.value.unshift(payload);
  } else if (editingId.value !== null) {
    users.value = users.value.map((user) => {
      if (user.id === editingId.value) {
        return payload;
      }
      return user;
    });
  }

  closeDialog();
};

const removeUser = (id: number) => {
  users.value = users.value.filter((user) => user.id !== id);
  if (page.value > pageCount.value) {
    page.value = pageCount.value;
  }
};

const toggleUserStatus = (user: UserItem) => {
  user.status = user.status === "启用" ? "禁用" : "启用";
};

const onSearch = () => {
  page.value = 1;
};

const logout = () => {
  window.alert("已退出登录");
};
</script>

<template>
  <v-app>
    <v-layout class="app-layout">
      <v-navigation-drawer permanent width="240" elevation="2">
        <v-list nav density="comfortable">
          <v-list-subheader>管理后台</v-list-subheader>
          <v-list-item
            v-for="item in menuItems"
            :key="item"
            :title="item"
            :active="item === activeMenu"
            @click="activeMenu = item"
          />
        </v-list>
      </v-navigation-drawer>

      <v-main class="bg-grey-lighten-4">
        <v-app-bar flat color="white" class="header-bar">
          <template #append>
            <div class="header-user">
              <v-avatar size="34">
                <v-img src="https://i.pravatar.cc/100?img=52" alt="当前用户头像" />
              </v-avatar>
              <span class="user-name">{{ currentUserName }}</span>
              <v-btn variant="text" color="error" @click="logout">退出</v-btn>
            </div>
          </template>
        </v-app-bar>

        <v-container fluid class="pa-6">
          <v-card>
            <v-card-title class="d-flex flex-wrap align-center ga-3">
              <span class="text-h6 font-weight-bold">用户列表</span>
              <v-spacer />
              <v-text-field
                v-model="searchKeyword"
                density="compact"
                variant="outlined"
                prepend-inner-icon="mdi-magnify"
                label="搜索姓名 / 电话 / 状态"
                hide-details
                class="search-input"
                @update:model-value="onSearch"
              />
              <v-btn color="primary" prepend-icon="mdi-plus" @click="openCreateDialog">
                新增用户
              </v-btn>
            </v-card-title>

            <v-divider />

            <v-data-table
              :headers="headers"
              :items="pagedUsers"
              :items-per-page="itemsPerPage"
              class="user-table"
              hover
            >
              <template #item.avatar="{ item }">
                <v-avatar size="36">
                  <v-img :src="item.avatar" :alt="item.name" />
                </v-avatar>
              </template>

              <template #item.status="{ item }">
                <v-chip :color="item.status === '启用' ? 'success' : 'grey'" size="small">
                  {{ item.status }}
                </v-chip>
              </template>

              <template #item.actions="{ item }">
                <div class="d-flex align-center ga-2">
                  <v-btn
                    size="small"
                    variant="text"
                    color="primary"
                    prepend-icon="mdi-pencil"
                    @click="openEditDialog(item)"
                  >
                    编辑
                  </v-btn>
                  <v-btn
                    size="small"
                    variant="text"
                    :color="item.status === '启用' ? 'warning' : 'success'"
                    @click="toggleUserStatus(item)"
                  >
                    {{ item.status === "启用" ? "禁用" : "启用" }}
                  </v-btn>
                  <v-btn
                    size="small"
                    variant="text"
                    color="error"
                    prepend-icon="mdi-delete"
                    @click="removeUser(item.id)"
                  >
                    删除
                  </v-btn>
                </div>
              </template>

              <template #bottom>
                <div class="d-flex justify-space-between align-center pa-4">
                  <div class="text-body-2 text-medium-emphasis">
                    共 {{ filteredUsers.length }} 条
                  </div>
                  <v-pagination
                    v-model="page"
                    :length="pageCount"
                    rounded="circle"
                    density="comfortable"
                  />
                </div>
              </template>
            </v-data-table>
          </v-card>
        </v-container>
      </v-main>
    </v-layout>

    <v-dialog v-model="dialogVisible" max-width="520">
      <v-card>
        <v-card-title class="text-h6 font-weight-bold">{{ dialogTitle }}</v-card-title>
        <v-card-text>
          <v-text-field
            v-model="userForm.avatar"
            label="头像 URL"
            variant="outlined"
            density="comfortable"
          />
          <v-text-field
            v-model="userForm.name"
            label="姓名"
            variant="outlined"
            density="comfortable"
          />
          <v-text-field
            v-model="userForm.phone"
            label="电话（11位手机号）"
            variant="outlined"
            density="comfortable"
          />
          <v-select
            v-model="userForm.status"
            :items="['启用', '禁用']"
            label="状态"
            variant="outlined"
            density="comfortable"
          />
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn variant="text" @click="closeDialog">取消</v-btn>
          <v-btn color="primary" :disabled="!isFormValid" @click="submitForm">保存</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<style scoped>
.app-layout {
  min-height: 100vh;
}

.header-bar {
  border-bottom: 1px solid rgb(0 0 0 / 8%);
}

.header-user {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-right: 8px;
}

.user-name {
  font-size: 14px;
  color: #333;
}

.search-input {
  width: 300px;
}

.user-table :deep(.v-data-table-footer) {
  display: none;
}
</style>
