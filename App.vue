<template>
  <View class="main">
    <!-- Status bar -->
    <Status-Bar background-color="#555" bar-style="light-content" />
    <!-- Top Bar -->
    <View class="top-bar">
      <Text class="top-bar-title">CRUD APP</Text>
      <Button
        v-if="isConnected"
        color="#00b609"
        class="connect-button"
        title="Connected"
        :on-press="checkForConnection"
      />
      <Button
        v-else
        color="#9c0000"
        class="connect-button"
        title="Disconnected"
        :on-press="checkForConnection"
      />
    </View>
    <!-- Content -->
    <View class="content">
      <!-- User inputs -->
      <Text class="subtitle-text">Insert, Update or Delete User</Text>
      <View class="user-panel">
        <Text-input
          class="user-input"
          placeholder="ID"
          style="flex: 1"
          :editable="false"
          v-model="selected_user.id"
        />
        <Text-input
          class="user-input"
          placeholder="First Name"
          style="flex: 4"
          v-model="selected_user.first_name"
        />
        <Text-input
          class="user-input"
          placeholder="Last Name"
          style="flex: 4"
          v-model="selected_user.last_name"
        />
        <Text-input
          class="user-input"
          placeholder="Email"
          style="flex: 9"
          v-model="selected_user.email"
        />
      </View>
      <View class="button-panel">
        <View class="button-wrapper">
          <Button
            color="#0076c5"
            class="button"
            title="Insert New"
            :on-press="insertUser"
            :disabled="selected_user.id != ''"
          />
        </View>
        <View class="button-wrapper">
          <Button
            color="#0076c5"
            class="button"
            title="Update"
            :on-press="updateUser"
            :disabled="selected_user.id == ''"
          />
        </View>
        <View class="button-wrapper">
          <Button
            color="#0076c5"
            class="button"
            title="Delete"
            :on-press="deleteUser"
            :disabled="selected_user.id == ''"
          />
        </View>
        <View class="button-wrapper">
          <Button
            color="#0076c5"
            class="button"
            title="Clear"
            :on-press="clearUser"
          />
        </View>
      </View>
      <!-- Query User -->
      <Text class="subtitle-text">User Table</Text>
      <View class="user-list-panel">
        <View class="user-list-item user-list-header">
          <Text class="color-white" style="flex: 1">ID</Text>
          <Text class="color-white" style="flex: 2">First Name</Text>
          <Text class="color-white" style="flex: 2">Last Name</Text>
          <Text class="color-white" style="flex: 4">Email</Text>
        </View>
        <Scroll-View>
          <Touchable-Opacity
            v-for="user in usersList"
            :key="user.id"
            class="user-list-item"
            :on-press="() => selectUser(user)"
          >
            <Text class="color-white" style="flex: 1">{{ user.id }}</Text>
            <Text class="color-white" style="flex: 2">{{ user.first_name }}</Text>
            <Text class="color-white" style="flex: 2">{{ user.last_name }}</Text>
            <Text class="color-white" style="flex: 4">{{ user.email }}</Text>
          </Touchable-Opacity>
        </Scroll-View>
      </View>
      <Button
        color="#0076c5"
        class="button"
        title="Refresh"
        :on-press="updateUserList"
      />
    </View>
  </View>
</template>

<script>
import axios from "axios";

const serverAddress = "http://192.168.1.6:3000";

export default {
  data: {
    isConnected: false,
    usersList: [],

    selected_user: {
      id: "",
      first_name: "",
      last_name: "",
      email: "",
    },

    delete_: { id: "" },
  },
  methods: {
    // Checks connections to the DB.
    checkForConnection() {
      let self = this;
      axios
        .get(serverAddress + "/ping", { timeout: 5 })
        .then(() => {
          self.isConnected = true;
        })
        .catch(() => {
          self.isConnected = false;
        });
    },

    // Updates the user list. (List displayed on screen by Vue).
    updateUserList() {
      axios.get(serverAddress + "/users").then((res) => {
        this.usersList = res.data;
      });
    },

    //Sets the pressed user from list into the selected user to allow Update and Delete.
    selectUser(user) {
      this.selected_user.id = user.id.toString();
      this.selected_user.first_name = user.first_name;
      this.selected_user.last_name = user.last_name;
      this.selected_user.email = user.email;
    },

    // HTTP POST with the user to insert
    insertUser() {
      axios
        .post(serverAddress + "/users/insert", { user: this.selected_user })
        .then((res) => {
          alert(res.data);
          this.updateUserList();
          this.clearUser();
        });
    },

    //  HTTP POST Updates the User with the selected ID with the selected values.
    updateUser() {
      axios
        .post(serverAddress + "/users/update", { user: this.selected_user })
        .then((res) => {
          alert(res.data);
          this.updateUserList();
          this.clearUser();
        });
    },

    //  HTTP POST Deletes the User with the selected ID.
    deleteUser() {
      axios
        .post(serverAddress + "/users/delete", { user: this.selected_user })
        .then((res) => {
          alert(res.data);
          this.updateUserList();
          this.clearUser();
        });
    },

    // Clears the user inputs.
    clearUser() {
      this.selected_user.id = "";
      this.selected_user.first_name = "";
      this.selected_user.last_name = "";
      this.selected_user.email = "";
    },
  },
  mounted() { 
    // When ready, start clock and check for conection every 3 seconds.
    this.checkForConnection();
    setInterval(this.checkForConnection, 3000);
  },
};
</script>



<style>

.main {
  background-color: #333;
  height: 100%;
}

.top-bar {
  width: 100%;
  height: 60px;
  background-color: #222;
  align-items: center;
  padding: 10px;
  flex-direction: row;
}

.top-bar-title {
  color: #ccc;
  font-size: 24px;
  flex: 1;
}

.connect-button {
  flex: 2;
  color: #9c0000;
}

.content {
  flex-direction: column;
  height: 90%;
  padding-left: 10px;
  padding-right: 10px;
  justify-content: space-around;
}

.subtitle-text {
  font-size: 24px;
  margin-bottom: 10px;
  margin-top: 10px;
  color: #dddddd;
}

/* Query Users */

.user-list-panel {
  width: 100%;
  height: 70%;
  margin-bottom: 10px;
  background-color: #222222;
}

.user-list-item {
  flex-direction: row;
  padding: 4px;
}

.user-list-header {
  background-color: #5f5f5f;
}

/* Insert User */

.user-panel {
  flex-direction: row;
}

.user-input {
  background-color: #646464;
  font-size: 16px;
  margin-left: 2px;
  margin-right: 2px;
  margin-bottom: 10px;
  border-color: #4e4e4e;
  border-width: 1px;
  border-radius: 2px;
  padding-left: 10px;
}

.button-panel {
  flex-direction: row;
}

.button-wrapper {
  flex: 1;
  margin-left: 2px;
  margin-right: 2px;
  align-self: stretch;
}

/* Misc */

.color-white {
  color: white;
}
</style>
