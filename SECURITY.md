<script src="https://www.gstatic.com/firebasejs/10.12.5/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.5/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.5/firebase-auth-compat.js"></script>

<script>
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyCDjZlqz2haR538suu1oc1xoAnlSnxI3po",
  authDomain: "happyuserer-45844.firebaseapp.com",
  databaseURL: "https://happyuserer-45844-default-rtdb.firebaseio.com",
  projectId: "happyuserer-45844",
  storageBucket: "happyuserer-45844.firebasestorage.app",
  messagingSenderId: "505209736403",
  appId: "1:505209736403:web:f7e6a34883b61632729826"
};

const FIREBASE_ADMIN_EMAIL = "dharmendraghathala@gmail.com";
const FIREBASE_ADMIN_UID = "5Hu7YJqITLQ8M0NndiwmgYxNNrz1";

firebase.initializeApp(FIREBASE_CONFIG);

const cloudDB = firebase.database();
let firebaseAdminUser = null;

firebase.auth().onAuthStateChanged(function(user) {
  firebaseAdminUser =
    user && user.uid === FIREBASE_ADMIN_UID ? user : null;
});

async function firebaseAdminLogin(password) {
  try {
    const result =
      await firebase.auth().signInWithEmailAndPassword(
        FIREBASE_ADMIN_EMAIL,
        password
      );

    if (result.user.uid !== FIREBASE_ADMIN_UID) {
      await firebase.auth().signOut();
      throw new Error("Admin UID mismatch");
    }

    firebaseAdminUser = result.user;
    return true;
  } catch (error) {
    console.error(error);
    return false;
  }
}

async function saveCloud(cloudState) {
  if (!firebaseAdminUser) {
    console.error("Firebase Admin login required");
    return false;
  }

  try {
    await cloudDB
      .ref("dhjiTournament/state")
      .set(cloudState);

    return true;
  } catch (error) {
    console.error("Cloud save failed:", error);
    return false;
  }
}

function startCloudSync(applyState) {
  cloudDB.ref("dhjiTournament/state").on("value", function(snapshot) {
    const data = snapshot.val();

    if (data && typeof applyState === "function") {
      applyState(data);
    }
  });
}

async function firebaseAdminLogout() {
  await firebase.auth().signOut();
  firebaseAdminUser = null;
}
</script>
