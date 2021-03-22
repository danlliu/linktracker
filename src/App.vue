<template>
  <div id="app" class="m-3">
    <div id="header" class="row p-3">
      <div class="col">
        <h1>link tracker</h1>
        <h6>by danlliu</h6>
      </div>
      <div class="col">

      </div>
    </div>

    <div id="body">
      <div class="row justify-content-between mb-4">

        <div class="col-2 dropdown">
          <button class="btn btn-outline-secondary dropdown-toggle" type="button" id="filterButton" data-toggle="dropdown"
                  aria-haspopup="true" aria-expanded="false">
            {{['All links', 'Upcoming', 'Today', 'Starred'][this.filterType]}}
          </button>
          <div class="dropdown-menu" aria-labelledby="filterButton">
            <button class="dropdown-item" type="button" v-on:click="filterType = 0">All links</button>
            <button class="dropdown-item" type="button" v-on:click="filterType = 1">Upcoming</button>
            <button class="dropdown-item" type="button" v-on:click="filterType = 2">Today</button>
            <button class="dropdown-item" type="button" v-on:click="filterType = 3">Starred</button>
          </div>
        </div>

        <div class="col-2">
          <button class="btn bi-plus-circle" id="addLink" data-toggle="modal" data-target="#addLinkForm"
                  style="font-size: 1.3rem"/>
        </div>

      </div>

      <transition-group name="list" class="lg" tag="div">
        <div v-for="(index, ix) in activeIndices" class="list-item row" :key="index">
          <div class="col-12 row">
            <div class="col-5">
              <h5>{{links[index].name}}</h5>
              <h6>time to time</h6>
            </div>
            <div class="col-5">
              <h5><a v-bind:href="links[index].link">{{links[index].link}}</a></h5>
              <h6 v-if="links[index].password == null">No password</h6>
              <h6 v-else>Password <b>{{links[index].password}}</b></h6>
            </div>
            <div class="col-2">
              <button class="btn" :class="links[index].starred ? 'bi-star-fill' : 'bi-star'"
                      style="color: gold;"/>
            </div>
          </div>
          <hr class="d-block col-12" v-if="ix !== activeIndices.length - 1"/>
        </div>
      </transition-group>

    </div>

    <div class="modal" tabindex="-1" id="addLinkForm">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Add a new link</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <form>
              <div class="form-group">
                <label for="nameInput">Event Title</label>
                <input type="text" class="form-control" id="nameInput" aria-describedby="nameHelp" required/>
                <small id="nameHelp" class="form-text text-muted">Enter the name of your event here. All event
                  data is stored on your browser's storage.</small>

                <label for="linkInput">Meeting Link</label>
                <input type="url" class="form-control" id="linkInput" aria-describedby="linkHelp" required/>
                <small id="linkHelp" class="form-text text-muted">Enter the link to your event here. All event
                  data is stored on your browser's storage.</small>

                <label for="pwdInput">Meeting Password</label>
                <input type="text" class="form-control form-control-sm" id="pwdInput" aria-describedby="pwdHelp"/>
                <small id="pwdHelp" class="form-text text-muted">Enter the password for the meeting link, if one
                exists.</small>

              </div>
              <hr/>
              <div class="form-group">

                <label for="datetimeInput">Meeting Date/Time</label>
                <input type="datetime-local" class="form-control" id="datetimeInput" aria-describedby="datetimeHelp"/>
                <small id="datetimeHelp" class="form-text text-muted">Enter the date and time of your event here.</small>

                <div class="form-check mb-1">
                  <input class="form-check-input" type="checkbox" value="" id="repeatCheck" v-model="formRepeating">
                  <label class="form-check-label" for="repeatCheck">
                    Repeating
                  </label>
                </div>

                <div v-if="formRepeating">
                  <div v-for="(day, idx) in ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday',
                  'Saturday']" :key="idx" class="form-check form-check-inline">
                    <input class="form-check-input" type="checkbox" id="inlineCheckbox1" value="">
                    <label class="form-check-label" for="inlineCheckbox1">{{day}}</label>
                  </div>
                </div>

              </div>

              <button type="submit" class="btn btn-primary">Add event</button>
            </form>
          </div>
          <div class="modal-footer">

          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>

  export default {
    name: 'App',
    components: {},
    data: () => {return {

      // form data
        formRepeating: false,

      // global data
        filterType: 0,
        links: [
          {
            name: "Staff Meeting",
            link: "https://meet.google.com/abc-defg-hij",
            starred: true
          },
          {
            name: "EECS 482 Lecture",
            link: "https://umich.zoom.us/j/1234567890",
            starred: false
          },
          {
            name: "EECS 481 Lecture",
            link: "https://umich.zoom.us/j/4824824824",
            starred: false
          },
          {
            name: "EECS 370 Discussion",
            link: "https://umich.zoom.us/j/3703703703",
            password: "123456",
            starred: false
          },
          {
            name: "370 Staff Meeting",
            link: "https://umich.zoom.us/j/0987654321",
            password: "567890",
            starred: false
          },
          {
            name: "EECS 370 Group OH",
            link: "https://umich.zoom.us/j/3703703704",
            starred: false
          }
        ]
      }},
    computed: {
      activeIndices: function() {

        let indices = [];

        switch (this.filterType) {
          case 0: // all
            for (let i = 0; i < this.links.length; ++i) {indices.push(i);}
            break;
          case 1: // upcoming
            for (let i = 0; i < this.links.length; ++i) {
             if (i % 2 === 0) {indices.push(i);}
            }
            break;
          default:
            for (let i = 0; i < this.links.length; ++i) {indices.push(i);}
            break;
        }

        return indices;
      }
    },

    mounted() {

    }

  }
</script>

<style>

  #header {
    position: fixed;
    width: 100%;
    top: 0;
    left: 0;
  }

  #body {
    margin: 96px 15% 0;
  }

  /* Transition */

  .list-item {
    transition: all 1s;
    display: inline-block;
    margin-bottom: 10px;
  }

  .list-enter, .list-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }

  .list-enter-active, .list-leave-active {
    transition: all 1s;
  }

  .list-move {
    transition: all 1s;
  }

  .list-leave-active {
    position: absolute;
    width: 70%;
  }

</style>
