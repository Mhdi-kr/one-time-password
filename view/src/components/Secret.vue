<template>
  <section>
    <div class="loader">
    </div>
    <main class="wrapper">
      <h1 class="glitch">{{ secret }}</h1>
    </main>
  </section>
</template>

<script>
const Crypto = require('crypto')
async function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
function makeid(length) {
  var result           = '';
  var characters       = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  var charactersLength = characters.length;
  for ( var i = 0; i < length; i++ ) {
    result += characters.charAt(Math.floor(Math.random() * charactersLength));
  }
  return result;
}
function bufferedSecret(secret) {
  if (typeof secret === 'string') {
    // Check if it's hex
    if (secret.match(/[0-9a-f]{40}/i)) {
      return Buffer.from(secret, 'hex');
    } else {
      // Looks like it's base64
      return Buffer.from(secret, 'base64');
    }
  }

  return secret;
}
export default {
  name: 'Secret',
  methods: {
    async calc(){
      let TIMEOFFSET = undefined
      let secret = bufferedSecret(makeid(5))
      let time = Math.floor(Date.now() / 1000) + (TIMEOFFSET || 0);
      let buffer = Buffer.allocUnsafe(8);
      buffer.writeUInt32BE(0, 0);
      buffer.writeUInt32BE(Math.floor(time / 30), 4);
      let hmac = Crypto.createHmac('sha1', secret);
      hmac = hmac.update(buffer).digest();
      let start = hmac[19] & 0x0F;
      hmac = hmac.slice(start, start + 4);
      let fullcode = hmac.readUInt32BE(0) & 0x7FFFFFFF;
      const chars = '23456789BCDFGHJKMNPQRTVWXY';
      let code = '';
      for (let i = 0; i < 5; i++) {
        code += chars.charAt(fullcode % chars.length);
        fullcode /= chars.length;
      }
      this.keys = code
      await sleep(3000)
      this.calc()
    }
  },
  beforeMount() {
    this.calc()
  },
  data: () => ({
    keys: 'asdasd'
  }),
  computed: {
    secret(){
      return this.keys.toUpperCase()
    }
  }
}
</script>

<style>
* {
  user-select: none;
  margin: 0;
  padding: 0;
}
.wrapper {
  height: 100vh;
  max-width: 100vw;
  background-color: black;
  display: flex;
  justify-content: center;
  align-items: center;
}
.glitch {
  font-size: 4rem;
  color: white;
  font-family: "Cascadia Code", serif;
}
.loader {
  position: absolute;
  height: 5px;
  background-color: yellow;
  animation: loading 3s linear infinite;
}
@keyframes loading {
  from {
    width: 0;
  }
  to {
    width: 100%;
  }
}
</style>


