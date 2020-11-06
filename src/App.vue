<template>
  <div id="app">
    <div id="container">
      <p id="logo">Aliens</p>
      <div id="space"></div>
      <button id="butt" v-on:click="getAliens(1)">Find</button>
    </div>
      <!-- <div v-for="item in alienArray" :key="item">>>>>>{{item}}</div> -->
    <yandex-map id="yaMap" class="ymap-container" :coords="coords" :zoom="3">
      <!-- <ymap-marker
        :coords="markerCoords"
        marker-id="1212313"
        hint-content="some hint"
        :balloon-template="balloonTemplate"
      /> -->
      <ymap-marker
       v-for="(item, index) in alienArray"
       :marker-id="random"
       :key="item.id"
       hint-content="some hint"
       :coords="item.coordinates"
       :balloon-template="balloonTemplate(index)"
       />
    </yandex-map>
  </div>
</template>

<script>
import { yandexMap, ymapMarker } from "vue-yandex-maps";
export default {
  name: "App",
  components: { yandexMap, ymapMarker },
  data: () => ({
    coords: [45.751244, 39.618423],
    alienArray: [],
    random: Math.random().toString(36).substring(2, 10),
  }),
   created() {
    this.getAliens(5);
  },
  methods: {
    balloonTemplate(i) {
      const alien = this.alienArray[i]
        return `
      <div id="baloonContainer">
        <div id="photoContainer">
          <img src="${alien.alienImage}">
          <img src="${alien.humanImage}">
        </div>
        <h3>${alien.title} ${alien.firstName} ${alien.lastName}</h3>
        <div id="innerContainer">
          <div class="infoContainer">
            <p class="strong">Gender: <span>${alien.gender}</span></p>
            <p class="strong">Age: <span>${alien.age}</span></p>
            <p class="strong">Nationality: <span>${alien.nationality}</span></p>
          </div>
          <div class="infoContainer">
            <p class="strong">Address:</p>
            <p class="pad">${alien.country}, ${alien.city}</p>
            <p class="pad">${alien.street}, ${alien.streetNumber}</p>
          </div>
          <div class="infoContainer">
            <p class="strong">Info:</p>
            <p class="strong pad">Phone: <span>${alien.phone}</span></p>
            <p class="strong pad">Cell phone: <span>${alien.cellPhone}</span></p>
            <p class="strong pad">Email: <span>${alien.email}</span></p>
          </div>
        </div>
      </div>
      `;
    },
    onClick(e) {
      console.log(e);
    },
    async fetchData() {
      const random = Math.random().toString(36).substring(2, 10);
      try {
        const response = await fetch('https://randomuser.me/api/');
        const data = await response.json()
        if (response.status === 200) {
          const person = {
            title: data.results[0].name.title,
            firstName: data.results[0].name.first,
            lastName: data.results[0].name.last,
            country: data.results[0].location.country,
            city: data.results[0].location.city,
            street: data.results[0].location.street.name,
            streetNumber: data.results[0].location.street.number,
            cellPhone: data.results[0].cell,
            phone: data.results[0].phone,
            email: data.results[0].email,
            gender: data.results[0].gender,
            humanImage: data.results[0].picture.large,
            age: data.results[0].dob.age,
            coordinates: [data.results[0].location.coordinates.latitude, data.results[0].location.coordinates.longitude],
            alienImage: `https://robohash.org/${random}?set=set2`,
            nationality: data.results[0].nat,
            id: random,
          }
          const cityUri = encodeURIComponent(person.city)
          const countryUri = encodeURIComponent(person.country)
          const streetUri = encodeURIComponent(person.street)
          const coordFetch = await fetch(`https://geocode-maps.yandex.ru/1.x/?apikey=127b876b-5803-484f-99f8-446bb1a9a619&format=json&geocode=${countryUri}+${cityUri}+${streetUri}+${person.streetNumber}`)
          // console.table(cityUri, person.city, countryUri, person.country)
          // const respLocation = await fetch(`https://api.opencagedata.com/geocode/v1/json?q=${countryUri}-${cityUri}&key=99e30435a9fa4beba5fe46479185267e`);
          const dataLocation = await coordFetch.json()
          // console.log(dataLocation.response.GeoObjectCollection.featureMember)
          console.log(dataLocation.response.GeoObjectCollection.featureMember.length)
          if (dataLocation.response.GeoObjectCollection.featureMember.length > 0) {
            const rightLocation = dataLocation.response.GeoObjectCollection.featureMember[0].GeoObject.Point.pos;
            const coordinatesArr = rightLocation.split(' ');
            person.coordinates = coordinatesArr.reverse()
            this.alienArray = [...this.alienArray, person]
          }
        }
      } catch (error) {
        console.log('fetch error', error)
      }
    },
    getAliens(times) {
      for (let i = 0; i < times; i++) {
        this.fetchData();
      }
      console.log('done')
    },
  },
};
</script>

<style>
/* @font-face kit by Fonts2u (https://fonts2u.com) */ @font-face {font-family:"Alien Resurrection";src:url("./assets/font/Alien_Resurrection.eot?") format("eot"),url("./assets/font/Alien_Resurrection.woff") format("woff"),url("./assets/font/Alien_Resurrection.ttf") format("truetype"),url("./assets/font/Alien_Resurrection.svg#") format("svg");font-weight:normal;font-style:normal;}
/* @font-face kit by Fonts2u (https://fonts2u.com) */ @font-face {font-family:"Space Age";src:url("./assets/font/space_age.eot?") format("eot"),url("./assets/font/space_age.woff") format("woff"),url("./assets/font/space_age.ttf") format("truetype"),url("./assets/font/space_age.svg#SpaceAge") format("svg");font-weight:normal;font-style:normal;}

#app {
  width: 100%;
  height: 100vh;
}

.ymap-container {
  height: 100%;
}

#container {
  width: 100%;
  height: 80px;
  background-color: black;
  background-blend-mode: lighten;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}

#logo {
  color: white;
  font-family: 'Alien Resurrection';
  font-size: 40px;
}

#space { 
  width: 200px;
}

#butt{
  text-decoration: none;
  background-color: white;
  border: none;
  border-radius: 15px;
  height: 30px;
  font-family: "Space Age";

}

#baloonContainer {
  display: flex;
  align-items: center;
  flex-direction: column;
  width: 210px;
  min-height: 330px;
}

#photoContainer {
  display: flex;
  flex-direction: row;
}

img {
  border-radius: 50%;
  width: 75px;
  height: 75px;
}

h3 {
  margin: 0px;
  margin-top: 12px;
}
p {
  margin-bottom: 3px;
  margin-top: 3px;
}

#innerContainer {
  display: flex;
  flex-direction: column;
  margin-top: 6px;
}

.infoContainer {
  border-bottom: 1px solid black;
  margin-top: 5px;
}

.strong {
  font-weight: bold;
}

.pad {
  margin-left: 5px;
}

span {
  font-weight: normal;
}
</style>
