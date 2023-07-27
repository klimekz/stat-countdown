import firebase from "firebase";

export default {
  setup() {
    const firebaseConfig = {
        apiKey: import.meta.env.VITE_API_KEY,
        authDomain: import.meta.env.VITE_AUTH_DOMAIN,
        projectId: "vue-nba-guess",
        storageBucket: import.meta.env.VITE_STORAGE_BUCKET,
        messagingSenderId: import.meta.env.VITE_MS_ID,
        appId: import.meta.env.VITE_APP_ID
    };
    const app = initializeApp(firebaseConfig)
    return {
      app,
    };
  },
};