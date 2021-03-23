<template>
  <div id="app" class="m-3" v-bind:class="darkMode && 'bg-dark'">

    <div id="header" class="row p-3" v-bind:class="darkMode && 'bg-dark'">
      <div class="col">
        <h1 v-bind:class="darkMode && 'text-light'">link tracker</h1>
        <h6 v-bind:class="darkMode && 'text-light'">by danlliu</h6>
      </div>
      <div class="col row justify-content-end align-content-end">
        <a href="https://github.com/danlliu/linktracker" target="_blank" rel="noreferrer noopener">v1.1.0</a>
      </div>
    </div>

    <div id="body">
      <div class="row justify-content-between align-items-center mb-4">

        <div class="col-2 dropdown">
          <button class="btn dropdown-toggle" v-bind:class="darkMode ? 'btn-outline-light' : 'btn-outline-secondary'"
                  type="button"
                  id="filterButton"
                  data-toggle="dropdown"
                  aria-haspopup="true" aria-expanded="false">
            {{['All links', 'Upcoming', 'Today', 'Starred'][this.filterType]}}
          </button>
          <div class="dropdown-menu" v-bind:class="darkMode && 'bg-secondary'"
               aria-labelledby="filterButton">
            <button class="dropdown-item" type="button" v-bind:class="darkMode && 'text-light'"
                    v-on:click="setFilter(0)">All
              links</button>
            <button class="dropdown-item" type="button" v-bind:class="darkMode && 'text-light'" v-on:click="setFilter(1)">Upcoming</button>
            <button class="dropdown-item" type="button" v-bind:class="darkMode && 'text-light'" v-on:click="setFilter(2)">Today</button>
            <button class="dropdown-item" type="button" v-bind:class="darkMode && 'text-light'" v-on:click="setFilter(3)">Starred</button>
          </div>
        </div>

        <div class="col-2">
          <button class="btn bi-plus-circle" id="addLink" data-toggle="modal" data-target="#addLinkForm"
                  style="font-size: 1.3rem" v-bind:class="darkMode && 'text-light'" onclick="$('#welcomeToast').toast('hide')"/>
        </div>

      </div>

      <transition-group name="list" tag="div" class="w-100">
        <div v-for="(item, ix) in activeIndices" class="list-item row" :key="links[item].idNum">
          <div class="col-12 row align-items-center">
            <div class="col-4">
              <h5 v-bind:class="darkMode && 'text-light'">{{links[item].name}}</h5>
              <h6 v-bind:class="darkMode && 'text-light'">
                <span v-if="links[item].time !== ''">{{formatTime(links[item].time)}}</span>
                <span v-else>no assigned time</span>
                <span v-if="links[item].repeating">, repeating
                  <span v-if="links[item].daysRepeating[0]">Su</span>
                  <span v-if="links[item].daysRepeating[1]">Mo</span>
                  <span v-if="links[item].daysRepeating[2]">Tu</span>
                  <span v-if="links[item].daysRepeating[3]">We</span>
                  <span v-if="links[item].daysRepeating[4]">Th</span>
                  <span v-if="links[item].daysRepeating[5]">Fr</span>
                  <span v-if="links[item].daysRepeating[6]">Sa</span>
                </span>
                <span v-else>
                  <span v-if="links[item].date !== ''">, {{formatDate(links[item].date)}}</span>
                  <span v-else>, no assigned date</span>
                </span>
              </h6>
            </div>
            <div class="col-6">
              <h5 v-bind:class="darkMode && 'text-light'"><a v-bind:href="links[item].link">
                <span class="full-link">{{links[item].link}}</span>
                <span class="short-link">Join meeting</span>
              </a></h5>
              <h6 v-bind:class="darkMode && 'text-light'" v-if="links[item].password === ''">No password</h6>
              <h6 v-bind:class="darkMode && 'text-light'" v-else>Password <b>{{links[item].password}}</b></h6>
            </div>
            <div class="col-1">
              <button class="btn" :class="links[item].starred ? 'bi-star-fill' : 'bi-star'" style="color: gold;"
              @click="links[item].starred = !links[item].starred;"/>
            </div>
            <div class="col-1">
              <button class="btn bi-x" style="font-size: 1.5rem" v-bind:class="darkMode && 'text-light'"
                      @click="deleteItem(item)"/>
            </div>
          </div>
          <hr class="col-12" v-bind:class="darkMode && 'bg-light'" v-if="ix !== activeIndices.length - 1"/>
        </div>
        <div v-bind:class="darkMode && 'text-light'" class="mt-5" :key="-1">
          <small>We store all your event data in your browser, so only your computer has access to your event
            details.</small>
          <br/>
          <small>&copy; 2021 Daniel Liu</small>
        </div>
      </transition-group>

    </div>

    <div class="modal fade" tabindex="-1" id="addLinkForm">
      <div class="modal-dialog modal-lg">
        <div class="modal-content" v-bind:class="darkMode && 'bg-dark'">
          <div class="modal-header">
            <h5 class="modal-title" v-bind:class="darkMode && 'text-light'">Add a new link</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close" v-on:click="clearForm">
              <span aria-hidden="true" v-bind:class="darkMode && 'text-light'">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <form v-on:submit.prevent="saveForm">
              <div class="form-group">
                <label for="nameInput" v-bind:class="darkMode && 'text-white'">Event Title</label>
                <input type="text" class="form-control" id="nameInput" aria-describedby="nameHelp"
                       v-model="formData.name" required autocomplete="off" v-bind:class="darkMode && 'bg-dark text-light'"/>
                <small id="nameHelp" class="form-text" v-bind:class="darkMode ? 'text-light' : 'text-muted'">Enter the
                  name of your event here. All event
                  data is stored on your browser's storage.</small>

                <label for="linkInput" v-bind:class="darkMode && 'text-white'">Meeting Link</label>
                <input type="url" class="form-control" id="linkInput" aria-describedby="linkHelp"
                       v-model="formData.link" required autocomplete="off" v-bind:class="darkMode && 'bg-dark text-light'"/>
                <small id="linkHelp" class="form-text" v-bind:class="darkMode ? 'text-light' : 'text-muted'">Enter the
                  link to your event here. All event
                  data is stored on your browser's storage.</small>

                <label for="pwdInput" v-bind:class="darkMode && 'text-white'">Meeting Password</label>
                <input type="text" class="form-control form-control-sm" id="pwdInput" aria-describedby="pwdHelp"
                       v-model="formData.password" autocomplete="off" v-bind:class="darkMode && 'bg-dark text-light'"/>
                <small id="pwdHelp" class="form-text" v-bind:class="darkMode ? 'text-light' : 'text-muted'">Enter the
                  password for
                  the meeting link, if one
                exists.</small>

              </div>
              <hr/>
              <div class="form-group">

                <label for="dateInput" v-bind:class="darkMode && 'text-white'">Meeting Date/Time</label>
                <div class="form-row">
                  <input type="date" class="form-control col-6" id="dateInput" autocomplete="false"
                         aria-describedby="datetimeHelp" v-model="formData.date" v-bind:class="darkMode && 'bg-dark text-light'"/>
                  <input type="time" class="form-control col-6" id="timeInput" autocomplete="false"
                         aria-describedby="datetimeHelp" v-model="formData.time" v-bind:class="darkMode && 'bg-dark text-light'"/>
                </div>
                <small id="datetimeHelp" class="form-text"
                       v-bind:class="darkMode ? 'text-light' : 'text-muted'">Enter the date and time of your event
                  here.
                If the event is repeating, enter the starting date and the time of the event.</small>

                <div class="form-check mb-1">
                  <input class="form-check-input" type="checkbox" value="" id="repeatCheck" v-model="formData.repeating" v-bind:class="darkMode && 'bg-dark text-light'">
                  <label class="form-check-label" for="repeatCheck" v-bind:class="darkMode && 'text-light'">
                    Repeating
                  </label>
                </div>

                <div v-if="formData.repeating">
                  <div v-for="(day, idx) in ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday',
                  'Saturday']" :key="idx" class="form-check form-check-inline">
                    <input class="form-check-input" type="checkbox" id="inlineCheckbox1" value=""
                           v-model="formData.daysRepeating[idx]" v-bind:class="darkMode && 'bg-dark text-light'">
                    <label class="form-check-label" for="inlineCheckbox1" v-bind:class="darkMode && 'text-light'">{{day}}</label>
                  </div>
                </div>

              </div>

              <button type="submit" class="btn btn-primary">Add event</button>
            </form>
          </div>
        </div>
      </div>
    </div>

    <div class="modal fade" tabindex="-1" id="delete">
      <div class="modal-dialog">
        <div class="modal-content" v-bind:class="darkMode && 'bg-dark'">
          <div class="modal-header">
            <h5 class="modal-title" v-bind:class="darkMode && 'text-light'">Delete event?</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close" v-bind:class="darkMode && 'text-light'">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <p v-bind:class="darkMode && 'text-light'">Are you sure you want to delete this event? You can't undo this
              .</p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">No, keep it.</button>
            <button type="button" class="btn btn-danger" @click="confirmDelete">Yes, BEGONE!</button>
          </div>
        </div>
      </div>
    </div>

    <div aria-live="polite" aria-atomic="true" style="position: relative; min-height: 200px;">
      <div style="position: fixed; top: 16px; right: 16px; z-index: 128">
        <div class="toast fade hide" id="welcomeToast"
             data-autohide="false">
          <div class="toast-header">
            <strong class="mr-auto">Getting Started</strong>
            <small>Just now</small>
            <button type="button" class="ml-2 mb-1 close" data-dismiss="toast" aria-label="Close"
                    onclick="localStorage.setItem('toastDismissed', 'yes')">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="toast-body">
            Welcome to link tracker! To get started, use the <i class="bi-plus-circle"/> icon to add a new event!
          </div>
        </div>
        <div class="toast fade hide" id="filterToast"
             data-autohide="false">
          <div class="toast-header">
            <strong class="mr-auto">Getting Started</strong>
            <small>Just now</small>
            <button type="button" class="ml-2 mb-1 close" data-dismiss="toast" aria-label="Close"
                    onclick="localStorage.setItem('filterDismissed', 'yes')">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="toast-body">
            <p>Now that you have an event added, you can click the button labeled "All links" to filter your events.</p>
            <p><b>Upcoming</b> shows all links that are scheduled to take place within the next 24 hours.</p>
            <p><b>Today</b> shows all links that are scheduled for today, whether before or after the current time.</p>
            <p><b>Starred</b> shows all links that you've marked as starred.</p>
          </div>
      </div>
      </div>
    </div>

    <a class="btn" v-bind:class="darkMode ? 'bi-moon text-light btn-dark' : 'bi-sun'"
            style="position: fixed; right: 16px; bottom: 16px; font-size: 1.5rem;" @click="flipDarkMode"/>

  </div>

</template>

<script>

  global.jQuery = require('jquery');
  var $ = global.jQuery;
  window.$ = $;

  export default {
    name: 'App',
    components: {},
    data: () => {return {

      // styling

        darkMode: false,

      // form data
        formData: {
          idNum: 0,
          name: "",
          link: "",
          password: "",
          date: "",
          time: "",
          repeating: false,
          daysRepeating: [],
          starred: false
        },

      // for making new object
        nextId: 0,

      // for deleting
        indexToRemove: -1,

      // global data
        filterType: 0,
        links: []
      }},
    computed: {
      activeIndices: function() {

        let indices = [];

        switch (this.filterType) {
          case 0: // all
            for (let i = 0; i < this.links.length; ++i) {indices.push(i);}
            break;
          case 1: // upcoming (next 24 h)
          {
            let datetimes = [];
            let now = new Date();
            let nowPlus24 = new Date();
            nowPlus24.setDate(nowPlus24.getDate() + 1);
            for (let i = 0; i < this.links.length; ++i) {

              if (this.links[i].repeating) {
                if (this.links[i].date === '') {
                  // no start date
                  if (this.links[i].daysRepeating[now.getDay()]) {
                    if (this.links[i].time === '') {
                      indices.push(i);
                      datetimes.push(0);
                    }
                    else {
                      let hour = parseInt(this.links[i].time.substr(0, 2));
                      let min = parseInt(this.links[i].time.substr(3, 2));
                      let currh = now.getHours();
                      let currm = now.getMinutes();
                      if (hour > currh) {
                        indices.push(i);
                        datetimes.push(0);
                      } else if (hour === currh && min >= currm) {
                        indices.push(i);
                        datetimes.push(0);
                      } else {
                        datetimes.push(0);
                      }
                    }
                  } else if (this.links[i].daysRepeating[nowPlus24.getDay()]) {
                    if (this.links[i].time === '') {
                      indices.push(i);
                      datetimes.push(0);
                    }
                    else {
                      let hour = this.links[i].time.substr(0, 2);
                      let min = this.links[i].time.substr(3, 2);
                      let currh = now.getHours();
                      let currm = now.getMinutes();
                      if (hour < currh) {
                        indices.push(i);
                        datetimes.push(0);
                      } else if (hour === currh && min <= currm) {
                        indices.push(i);
                        datetimes.push(0);
                      } else {
                        datetimes.push(0);
                      }
                    }
                  } else {
                    datetimes.push(0);
                  }
                } else {
                  // start date
                  if (this.links[i].time === '') {
                    let parsed = this.splitDate(this.links[i].date);
                    let itemDate = new Date(parsed[0], parsed[1] - 1, parsed[2]);
                    if (itemDate <= nowPlus24) {
                      if (this.links[i].daysRepeating[now.getDay()] ||
                              this.links[i].daysRepeating[nowPlus24.getDay()]) {
                        indices.push(i);
                        datetimes.push(0);
                      } else {
                        datetimes.push(0);
                      }
                    } else {
                      datetimes.push(0);
                    }
                  }
                  else {
                    let parsed = this.splitDate(this.links[i].date);
                    let itemDate = new Date(parsed[0], parsed[1] - 1, parsed[2]);
                    let hour = parseInt(this.links[i].time.substr(0, 2));
                    let min = parseInt(this.links[i].time.substr(3, 2));
                    let currh = now.getHours();
                    let currm = now.getMinutes();
                    if (itemDate <= nowPlus24) {
                      if (this.links[i].daysRepeating[now.getDay()]) {
                        if (hour > currh) {
                          indices.push(i);
                          datetimes.push(0);
                        }
                        else if (hour === currh && min > currm) {
                          indices.push(i);
                          datetimes.push(0);
                        }
                        else {
                          datetimes.push(0);
                        }
                      }
                      else if (this.links[i].daysRepeating[nowPlus24.getDay()]) {
                        if (hour < currh) {
                          indices.push(i);
                          datetimes.push(0);
                        }
                        else if (hour === currh && min < currm) {
                          indices.push(i);
                          datetimes.push(0);
                        }
                        else {
                          datetimes.push(0);
                        }
                      }
                      else {
                        datetimes.push(0);
                      }
                    }
                    else {
                      datetimes.push(0);
                    }
                  }
                }
                continue;
              }

              if (this.links[i].date !== '') {
                if (this.links[i].time === '') {
                  let parsed = this.splitDate(this.links[i].date);
                  let itemDate = new Date(parsed[0], parsed[1] - 1, parsed[2]);
                  if (itemDate >= now.getTime()) {
                    indices.push(i);
                    datetimes.push(itemDate);
                  } else {
                    datetimes.push(0);
                  }
                } else {
                  let parsed = this.splitDate(this.links[i].date);
                  let hour = this.links[i].time.substr(0, 2);
                  let min = this.links[i].time.substr(3, 2);
                  let d = new Date(parsed[0], parsed[1] - 1, parsed[2], hour, min);
                  if (d >= now.getTime() && d <= nowPlus24.getTime()) {
                    indices.push(i);
                    datetimes.push(d);
                  } else {
                    datetimes.push(0);
                  }
                }
              } else {
                datetimes.push(0);
              }
            }
            console.log(datetimes);
            indices.sort((x, y) => datetimes[x] - datetimes[y]);
            console.log(indices);
          }
            break;
          case 2: // today
          {
            let datetimes = [];
            let now = new Date();
            let nowstring = now.getFullYear() + "-" + (now.getMonth() < 9 ? '0' : '') + (now.getMonth() + 1) + "-" +
                    (now.getDate() < 10 ? '0' : '') + now.getDate();
            console.log(nowstring);
            for (let i = 0; i < this.links.length; ++i) {

              if (this.links[i].repeating) {
                if (this.links[i].date === '') {
                  if (this.links[i].daysRepeating[now.getDay()]) {
                    indices.push(i);
                    datetimes.push(0);
                  } else {
                    datetimes.push(0);
                  }
                } else {
                  let parsed = this.splitDate(this.links[i].date);
                  let itemDate = new Date(parsed[0], parsed[1] - 1, parsed[2]);
                  if (itemDate <= now && this.links[i].daysRepeating[now.getDay()]) {
                    indices.push(i);
                    datetimes.push(0);
                  } else {
                    datetimes.push(0);
                  }
                }
                continue;
              }

              console.log(this.links[i].date);
              if (this.links[i].date === nowstring) {
                if (this.links[i].time === '') {
                  let itemDate = Date.parse(this.links[i].date);
                  indices.push(i);
                  datetimes.push(itemDate);
                } else {
                  let d = Date.parse(this.links[i].date + "T" + this.links[i].time);
                  indices.push(i);
                  datetimes.push(d);
                }
              } else {
                datetimes.push(0);
              }
            }
            console.log(datetimes);
            indices.sort((x, y) => datetimes[x] - datetimes[y]);
            console.log(indices);
          }
            break;
          case 3: // starred
            for (let i = 0; i < this.links.length; ++i) {
              if (this.links[i].starred) {indices.push(i);}
            }
            break;
          default:
            for (let i = 0; i < this.links.length; ++i) {indices.push(i);}
            break;
        }

        return indices;
      }

    },
    methods: {

      setFilter: function(num) {
        this.filterType = num;
        $('#filterToast').toast('hide');
        localStorage.setItem("filterDismissed", "yes");
      },

      saveData: function() {
        localStorage.setItem("links", JSON.stringify(this.links));
        localStorage.setItem("nextId", this.nextId.toString());
      },

      saveForm: function() {
        // i mean you probably shouldn't be using http for a meeting but whatever
        if (this.formData.link.substr(0, 7) !== "http://" && this.formData.link.substr(0, 8) !== "https://") {
          this.formData.link = "https://" + this.formData.link;
        }

        this.formData.idNum = this.nextId++;
        this.nextId %= 1048576;

        this.links.push(this.formData);
        this.formData = {
          idNum: 0,
          name: "",
          link: "",
          password: "",
          date: "",
          time: "",
          repeating: false,
          daysRepeating: [],
          starred: false
        };

        $('#addLinkForm').modal('hide');

        if (localStorage.getItem('filterDismissed') == null) $('#filterToast').toast('show');

        this.saveData();
      },

      clearForm: function() {
        this.formData = {
          idNum: 0,
          name: "",
          link: "",
          password: "",
          date: "",
          time: "",
          repeating: false,
          daysRepeating: [],
          starred: false
        };
      },

      deleteItem: function(index) {
        this.indexToRemove = index;
        $('#delete').modal('show');
      },

      confirmDelete: function() {
        if (this.indexToRemove !== -1) {
          $('#delete').modal('hide');
          this.links.splice(this.indexToRemove, 1);
          this.saveData();
          this.indexToRemove = -1;
        }
      },

      formatTime: function(t) {
        let dateObj = new Date("2021-01-01T"+t);
        console.log(dateObj);
        return dateObj.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
      },

      formatDate: function(d) {
        let parsed = this.splitDate(d);
        let dateObj = new Date(parsed[0], parsed[1] - 1, parsed[2]);
        return dateObj.toLocaleDateString();
      },

      splitDate: function(d) {
        // YYYY-MM-DD
        let year = parseInt(d.substr(0, 4));
        let month = parseInt(d.substr(5, 2));
        let day = parseInt(d.substr(8, 2));
        return [year, month, day]
      },

      flipDarkMode: function() {
        this.darkMode = !this.darkMode;
        localStorage.setItem('darkMode', this.darkMode ? "yes" : "no");
        if (this.darkMode) {
          $('body').addClass('bg-dark');
        } else {
          $('body').removeClass('bg-dark');
        }
      }

    },

    mounted() {
      console.log('mounted!');
      if (localStorage.getItem('links') != null) {
        this.links = JSON.parse(localStorage.getItem('links'));
        this.nextId = parseInt(localStorage.getItem('nextId'));
      } else {
        this.links = [];
        this.nextId = 0;
      }

      if (localStorage.getItem('darkMode') == null) {
        localStorage.setItem('darkMode', 'no');
      } else if (localStorage.getItem('darkMode') === 'yes') {
        this.darkMode = true;
      }

      if (this.darkMode) {
        $('body').addClass('bg-dark');
      } else {
        $('body').removeClass('bg-dark');
      }

      if (localStorage.getItem('toastDismissed') == null) $('#welcomeToast').toast('show');
      window.addEventListener('beforeunload', () => {this.saveData();});
    }

  }
</script>

<style>

  #header {
    position: fixed;
    width: 100%;
    top: 0;
    left: 0;
    background-color: white;
    z-index: 64;
  }

  #body {
    margin-top: 128px;
  }

  /* Transition */

  .list-item {
    transition: all 1s;
    margin-bottom: 10px;
  }

  .list-enter, .list-leave-to {
    opacity: 0;
  }

  .list-enter-active, .list-leave-active {
    transition: all 1s;
  }

  .list-move {
    transition: all 1s;
  }

  .list-leave-active {
    position: absolute;
  }

  /* show link on wide screen but join on small screen */

  @media only screen and (min-width: 1100px) {
    .full-link {
      display: inherit;
    }
    .short-link {
      display: none;
    }

    #body {
      margin-left: 15%;
      margin-right: 15%;
    }

    .list-item {
      width: 70vw;
    }

    .list-leave-active {
      width: 70% !important;
    }
  }

  @media only screen and (max-width: 1099px) {
    .full-link {
      display: none;
    }
    .short-link {
      display: inherit;
    }

    #body {
      margin-left: 5%;
      margin-right: 5%;
    }

    .list-item {
      width: 90vw;
    }

    .list-leave-active {
      width: 90% !important;
    }
  }

</style>
