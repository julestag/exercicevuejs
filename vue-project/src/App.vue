<template>
  <div>
    <div class="header">
      <div class="menu-container">
        <div class="menu-button" @click="toggleMenu">
          <div class="bar">
           <div class="bar"></div>
          </div>
         
          <div class="bar"></div>
        </div>
      </div>
      <div class="close-button" v-show="menuOpen" @click="toggleMenu"></div>
      <img src="./logo.png" alt="Logo" class="logo">
    </div>
    <div class="menu" :class="{ 'active': menuOpen }">
      <!-- Contenu du menu -->
      <div class="menu-content">
        <h3>Menu</h3>
        <!-- Ajout de la croix pour fermer le menu -->
        <div class="close-button" @click="toggleMenu">
          <div class="close-icon">&nbsp &nbsp X</div>
        </div>
      </div>
    </div>
    <div class="dashboard">
      <div class="carrer">
        <h2 class="title">Statistique d'occupation du bâtiment : </h2>
      </div>
      <br>
      <br>
      <br>
      <br>
      <div class="container">
        <div class="floor-buttons">
          <h4
            v-for="(building, index) in buildings"
            :key="index"
            class="building"
          >
            <div
              v-for="(floor, floorIndex) in building.floors"
              :key="floorIndex"
              class="floor"
            >
              <h4
                class="floor-button"
                @click="toggleFloor(floorIndex)"
              >
                {{ floor.name }}
              </h4>
            </div>
          </h4>
        </div>
        <div class="room-details">
          <ul class="room-list">
            <li
              v-for="room in selectedRooms"
              :key="room.name"
              class="room"
            >
              <span class="room-bubble">
                <span class="room-name">n°{{ room.name }} <br></span>
                <br>
                <span class="separator">statut d'occupation : </span>
                <span class="occupation">{{ getOccupation(room.occupation) }}</span>
              </span>
            </li>
          </ul>
        </div>
        <br>
        <br>
            <div class="room-summary">
      <div>Total des pièces : {{ totalRooms }}</div>
      <div>Pièces occupées : {{ occupiedRooms }}</div>
      <div>Pièces libres : {{ freeRooms }}</div>
      <div>Pièces non définies : {{ undefinedRooms }}</div>
      <div class="progress-bar">
        <div class="progress-bar-inner" :style="{ width: occupiedPercentage + '%' }"></div>
      </div>
      <div class="progress-bar">
        <div class="progress-bar-inner" :style="{ width: freePercentage + '%' }"></div>
      </div>
      <div class="progress-bar">
        <div class="progress-bar-inner" :style="{ width: undefinedPercentage + '%' }"></div>
      </div>
    </div>
      </div>
    </div>
  </div>
</template>

<script>
import geographicContext from './gegraphicContext_space.json';
import roomEndpoints from './ROOM_control_endpoin_list.json';

export default {
  data() {
    return {
      buildings: [],
      selectedFloor: null,
      menuOpen: false
    };
  },
  created() {
    this.loadBuildingData();
  },
  methods: {
    loadBuildingData() {
      this.buildings = geographicContext.children.map(building => ({
        name: building.name,
        floors: building.children.map(floor => ({
          name: floor.name,
          rooms: floor.children.map(room => {
            const matchedEndpoint = roomEndpoints.find(endpoint => endpoint.dynamicId === room.dynamicId);
            return {
              name: room.name,
              occupation: matchedEndpoint ? matchedEndpoint.currentValue : "UNDEFINED"
            };
          })
        }))
      }));
    },
    getOccupation(value) {
      if (value === true) {
        return "Occupée";
      } else if (value === false) {
        return "Libre";
      } else {
        return "Undefined";
      }
    },
    toggleFloor(floorIndex) {
      this.selectedFloor = this.selectedFloor === floorIndex ? null : floorIndex;
    },
    toggleMenu() {
      this.menuOpen = !this.menuOpen;
    }
  },
  computed: {
    selectedRooms() {
      if (this.selectedFloor !== null && this.buildings.length > 0) {
        return this.buildings[0].floors[this.selectedFloor].rooms;
      }
      return [];
    },
    totalRooms() {
      return this.buildings.flatMap(building => building.floors.flatMap(floor => floor.rooms)).length;
    },
    occupiedRooms() {
      return this.selectedRooms.filter(room => room.occupation === true).length;
    },
    freeRooms() {
      return this.selectedRooms.filter(room => room.occupation === false).length;
    },
    undefinedRooms() {
      return this.selectedRooms.filter(room => room.occupation === "UNDEFINED").length;
    },
    occupiedPercentage() {
      return (this.occupiedRooms / this.totalRooms) * 100;
    },
    freePercentage() {
      return (this.freeRooms / this.totalRooms) * 100;
    },
    undefinedPercentage() {
      return (this.undefinedRooms / this.totalRooms) * 100;
    }
  }
};
</script>

<style>
body {
  background-color: #000;
  color: #fff;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
}

.header {
  background-color: #000000;
  color: #fff;
  padding: 40px;
}

.menu-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.menu-button {
  cursor: pointer;
  display: flex;
  align-items: center;
}

.bar {
  width: 30px;
  height: 3px;
  background-color: white;
  margin: 8px 0;
  transition: transform 0.3s, opacity 0.3s;
}

.close-button {
  width: 30px;
  height: 30px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-icon {
  color: #fff;
  font-size: 24px;
}

.menu {
  width: 200px;
  height: 100vh;
  position: fixed;
  top: 0;
  left: -200px;
  background-color: #F39422;
  opacity: 0.9;
  transition: left 0.3s;
}

.menu.active {
  left: 0;
}

.dashboard {
  margin-left: 70px;
  padding: 8px;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  width: 1300px;
  height: 575px;
  color: #000;
  border-radius:10px;
}

.title {
  margin: 0;
  font-size: 24px;
}

.container {
  display: flex;
  border-color: black;
  padding: 20px;
}

.floor-buttons {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 30%;
  background-color:#FFF;
}

.floor-button {
  background-color: #000;
  color: #fff;
  border: 1px solid #000;
  padding: 10px 20px;
  margin: 10px 0;
  cursor: pointer;
}

.room-details {
  background-color: #fff;
  color: #000;
  border: 2px solid #000;
  padding: 10px;
  margin-left: 20px;
    margin-right: 40px;
  width: 40%;
  height: 400px;
  overflow-y: auto;
  background-color:black;
    border-radius:10px;

}

.room-list {
  list-style: none;
  padding: 0;
}

.room {
  margin-bottom: 10px;
}

.room-bubble {
  background-color: #F39422;
  color: #000;
  border-radius: 15px;
  padding: 10px;
  display: inline-block;
}

.logo {
  width: 200px;
  margin-left: 70px;
}

.room-summary {
  margin-top: 20px;
}

.progress-bar {
  margin-top: 10px;
  width: 100%;
  height: 20px;
  background-color: #ccc;
  border-radius: 5px;
}

.progress-bar-inner {
  height: 100%;
  background-color: #F39422;
  border-radius: 5px;
}
</style>
