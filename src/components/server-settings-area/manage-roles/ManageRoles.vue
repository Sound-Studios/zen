<template>
  <div class="container manage-roles">
    <div class="inner-container">
      <div class="description">
        <div class="material-icons">info</div>
        Manage Roles
      </div>
      <div class="notice">
        Manage your roles.
      </div>
      <SelectedRolesPage
        v-if="selectedRoleID"
        :roleID="selectedRoleID"
        @close="selectedRoleID = null"
      />
      <div class="box" v-if="!selectedRoleID">
        <CustomButton
          class="button"
          name="Create New Role"
          icon="add"
          @click="createRole"
        />

        <div class="role-list">
          <Draggable
            :animation="200"
            ghost-class="ghost"
            :delay="$isMobile ? 400 : 0"
            v-model="roles"
            @end="onDragEnd"
          >
            <RoleTemplate
              v-for="role in roles"
              :key="role.id"
              :role="role"
              @click.native="selectedRoleID = role.id"
            />
          </Draggable>
          <!-- Default role always stays at the bottom. -->
          <RoleTemplate
            :role="defaultRole"
            @click.native="selectedRoleID = defaultRole.id"
          />
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import CustomInput from "@/components/CustomInput.vue";
import Draggable from "vuedraggable";
import { ServersModule } from "@/store/modules/servers";
import CustomButton from "@/components/CustomButton.vue";
import RoleTemplate from "./RoleTemplate.vue";
import ContextMenu from "@/components/ContextMenu.vue";
import SelectedRolesPage from "./SelectedRolesPage.vue";
import { createServerRole, updateRolePosition } from "@/services/rolesService";
import { ServerRolesModule } from "@/store/modules/serverRoles";
import ServerRole from "@/interfaces/ServerRole";
@Component({
  components: {
    CustomInput,
    CustomButton,
    RoleTemplate,
    ContextMenu,
    Draggable,
    SelectedRolesPage
  }
})
export default class ManageRoles extends Vue {
  selectedRoleID: string | null = null;
  createRequestSent = false;

  onDragEnd(event: any) {
    const newIndex = event.newIndex;
    const role = this.roles[newIndex];
    const sendObj = { roleID: role.id, order: newIndex };
    updateRolePosition(this.serverID, sendObj);
  }

  createRole() {
    if (this.createRequestSent) return;
    this.createRequestSent = true;
    createServerRole(this.serverID)
      .then(res => {
        ServerRolesModule.AddServerRole(res);
        this.selectedRoleID = res.id;
      })
      .finally(() => {
        this.createRequestSent = false;
      });
  }

  get server() {
    return ServersModule.servers[this.serverID];
  }
  get roles() {
    return ServerRolesModule.sortedServerRolesArr(this.serverID).filter(
      r => !r.default
    );
  }
  set roles(roles: ServerRole[]) {
    const orderedRoles = roles.map((r, index) => {
      return { ...r, order: index };
    });
    // to obj
    const obj: any = {};
    for (let i = 0; i < orderedRoles.length; i++) {
      const item = orderedRoles[i];
      obj[item.id] = item;
    }
    this.defaultRole && (obj[this.defaultRole?.id] = this.defaultRole);
    ServerRolesModule.AddServerRoles({
      roles: obj,
      serverID: this.serverID
    });
  }
  get defaultRole() {
    return ServerRolesModule.defaultServerRole(this.serverID);
  }
  get serverID() {
    return this.$route.params.server_id;
  }
}
</script>

<style lang="scss" scoped>
.ghost {
  opacity: 0;
}
.container {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
  flex: 1;
}
.inner-container {
  display: flex;
  flex: 1;
  flex-direction: column;
  overflow: hidden;
}
.description {
  display: flex;
  align-items: center;
  align-content: center;
  margin-left: 10px;
  margin-top: 10px;
  .material-icons {
    margin-right: 5px;
  }
}
.role-list {
  display: flex;
  flex-direction: column;
  overflow: auto;
  padding: 10px;
}

.notice {
  color: rgba(255, 255, 255, 0.6);
  margin-left: 40px;
}
.box {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  height: 100%;
}
.button {
  align-self: flex-start;
  border: none;
  color: white;
  margin-left: 10px;
  background: rgba(255, 255, 255, 0.03);
}
</style>