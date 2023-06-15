<template>
    <div class="relative overflow-x-auto mt-[10vh] md:mx-10">
        <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
            <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
                <tr>
                    <th scope="col" class="px-6 py-3">
                        Ime Korisnika
                    </th>
                    <th scope="col" class="px-6 py-3">
                        Vrsta termina
                    </th>
                    <th scope="col" class="px-6 py-3">
                        Vrijeme
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="app in appointments" :key="app.uid"
                    class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
                    <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                        {{ app.user }}
                    </th>
                    <td class="px-6 py-4">
                        {{ app.type }}
                    </td>
                    <td class="px-6 py-4">
                        {{ new Date(app.time).toLocaleString() }}
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

</template>

<script>
import { getDocs, query, where, collection } from "firebase/firestore";
import { db } from "../main.js";
export default {
    data() {
        return {
            user: false,
            appointments: [],
        }
    },
    mounted() {
        this.user = this.$store.state.user.loggedIn;
        if (!this.user) {
            this.$router.push('/auth');
        }
        this.getAppointments();
        console.log(this.$store.state.user.data)
    },
    methods: {
        async getAppointments() {
            let q;
            if (this.$store.state.user.data.email === 'admin@gmail.com') {
                q = query(collection(db, "termin"));
            } else {
                q = query(collection(db, "termin"), where("user_id", "==", this.$store.state.user.data.email));
            }

            const querySnapshot = await getDocs(q);
            querySnapshot.forEach((doc) => {
                this.appointments.push({
                    uid: doc.id,
                    user: doc.data().displayName ? doc.data().displayName : doc.data().user_id,
                    type: doc.data().app_type,
                    time: doc.data().time.seconds * 1000
                });
            });
            this.appointments.sort((a, b) => {
                return b.time - a.time;
            });
        },
    }
}
</script>