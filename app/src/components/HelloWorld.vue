<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-show="castSession == null" @click="initializeCast">Init Cast</button>
    <button v-show="castEnabled" @click="requestCastSession">requestCastSession</button>
    <p v-if="castSession != null">Connected <button @click="stopCastSession">stopCastSession</button></p>
    <p v-else>Not connected</p>
    <button v-show="castSession != null" @click="sendDefaultMessage">Send Message</button>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Chromecast Sender App',
      castSession: null,
      castEnabled: false,
      count: 0,
      namespace: "urn:x-cast:com.wearemoka.chromecastexample",
    }
  },
  methods: {
      initializeCast () {
        var self = this;
        // var appId = chrome.cast.media.DEFAULT_MEDIA_RECEIVER_APP_ID;
        var appId = "356D62CE"; // replace with your own APP_ID
        var apiConfig = new chrome.cast.ApiConfig(new chrome.cast.SessionRequest(appId), this.sessionListener, function receiverListener (receiverAvailable) {
            // receiverAvailable is a string.
            // "available" = at least one chromecast device is available
            // "unavailable" = No chromecast devices available
            // You can use this to determine if you want to show your chromecast icon
            if (receiverAvailable == "available") {
              console.log("receiverAvailable available");
              self.castEnabled = true; // to show or hide icon
            } else {
              console.log("receiverAvailable unavailable");
            }
          });

        // initialize chromecast, this must be done before using other chromecast features
        chrome.cast.initialize(apiConfig, function () {
            // Initialize complete
          }, function (err) {
            // Initialize failure
            console.log("Initialize failure");
            console.log(err);
          }
        );
      },
      sessionListener (session) {
        // The session listener is only called under the following conditions:
        // * will be called shortly chrome.cast.initialize is run
        // * if the device is already connected to a cast session
        // Basically, this is what allows you to re-use the same cast session
        // across different pages and after app restarts
        if (session != null && this.castSession == null) {
          this.castSession = session;
        }
      },
      requestCastSession () {
        // This will open a native dialog that will let
        // the user choose a chromecast to connect to
        // (Or will let you disconnect if you are already connected)
        var self = this;
        chrome.cast.requestSession(function (session) {
          // Got a session!
          self.castSession = session;

          let message = self.infoToCast();

          self.sendCastMessage(message);
        }, function (err) {
          // Failed, or if err is cancel, the dialog closed
          console.log("Failed to stop cast session1 " + JSON.stringify(err));
        });
      },
      stopCastSession () {
        var self = this;
        // Also stop the session (if)
        if (this.castSession) {
          this.castSession.stop(function () {
            // Success
            console.log("Stopped cast session");
            self.castSession = null;
          }, function (err) {
            // Fail
            self.castSession = null;
            console.log("Failed to stop cast session2 " + JSON.stringify(err));
          });
        }
      },
      sendCastMessage(message) {
        if (this.castSession != null) {
          console.log("cast session active " + this.namespace);
          this.castSession.sendMessage(this.namespace, message, this.onCastSuccess.bind(message), this.onCastError.bind(message));
        }
        else {
          console.log("session null to send message");
          var self = this;
          chrome.cast.requestSession(function (session) {
            self.castSession = session;
            self.sessionListener(session);
            self.castSession.sendMessage(self.namespace, message, self.onCastSuccess.bind(message), self.onCastError.bind(message));
          }, this.onCastError);
        }
      },
      onCastSuccess(message) {
        console.log('onCastSuccess: ' + JSON.stringify(message));
      },
      onCastError(message) {
        console.log('onCastError: ' + JSON.stringify(message));
      },
      infoToCast() {
        this.count++;
        return {
          "myData": this.msg + this.count,
        }
      },
      sendDefaultMessage(){
        const message = this.infoToCast();
        this.sendCastMessage(message);
      },
    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
