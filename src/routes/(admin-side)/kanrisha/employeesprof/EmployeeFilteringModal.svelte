<script>
    import {db} from '$lib/firebase/client.js';
    import { collection, where, query, getDocs, orderBy  } from 'firebase/firestore';

    let filteringModal = false; 
    let searchInput = "";
    let searchInputAddress = "";
    let searchResults = [];
    let searchResultOne = [];
    let searchResultTwo = [];
    let searchResultThree = [];

    // export async function getAllClients(clientWithLoans){
    //     let qOne;
    //     if (clientWithLoans == true) {
    //         qOne = query(collection(db, "clientinfo"), orderBy("status", "desc"),orderBy("clientNumber", "desc"), where("status", "!=", "No Loan") );
    //     } else {
    //         qOne = query(collection(db, "clientinfo"), orderBy("clientNumber", "desc"));
    //     }
    //     const querySnapshot = await getDocs(qOne);
    //     querySnapshot.forEach((doc) => {
    //         searchResults = querySnapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
    //     });
    // }

    function resetAddUserInput () {
        searchInput = "";
        searchResults = [];
    }

    async function searchClient() {
        searchResults = [];
        const qOne = query(collection(db, 'clientinfo'), where("firstname", "==", searchInput));
        const querySnapshotOne = await getDocs(qOne);
        querySnapshotOne.forEach((doc) => {
            searchResultOne = querySnapshotOne.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
        });
        const qTwo = query(collection(db, "clientinfo"), where("lastname", "==", searchInput));
        const querySnapshotTwo = await getDocs(qTwo);
        querySnapshotTwo.forEach((doc) => {
            searchResultTwo = querySnapshotTwo.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
        });
        const qThree = query(collection(db, "clientinfo"), where("address", "==", searchInput));
        const querySnapshotThree = await getDocs(qThree);
        querySnapshotThree.forEach((doc) => {
            searchResultThree = querySnapshotThree.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
        });
        searchResults = searchResultOne.concat(searchResultTwo,searchResultThree);
        searchInput = "";
    }

</script>

<input type="checkbox" bind:checked={filteringModal} id="filter" class="modal-toggle" />
    <div class="modal">
        <div class="modal-box  w-1/2">

            <form class=" bg-white rounded-lg w-full">
                <!-- Modal header -->
                <div class="flex justify-center items-center p-4 rounded-t">
                    <h3 class="font-semibold text-gray-900">
                        Filter by Name:
                    </h3>
                    <input type="search" bind:value={searchInput} class="w-full border-0 border-b-2">
                </div>
                
                <div class="flex justify-center items-center p-4 rounded-t border-b">
                    <h3 class="font-semibold text-gray-900">
                        Filter by Address:
                    </h3>
                    <input type="search" bind:value={searchInputAddress} class="w-full border-0 border-b-2">
                </div>
             
                <div class="flex justify-left items-center p-2 rounded-t">
                    <h3 class="font-semibold text-gray-900 px-3">
                        Filter Role:
                    </h3>
                    <input type="radio" id="asc" name="role" value="cashier">
                    <label for="cashier" class="px-2">Cashier</label>
                    <input type="radio" id="desc" name="role" value="collector">
                    <label for="collector" class="px-2">Collector</label>
                </div>

                <div class="modal-action">    
                    <button type="submit" class="btn border-transparent bg-blue-600">Apply</button>
                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                    <label for="filter" on:click={resetAddUserInput} class="btn border-transparent bg-red-600">Cancel</label>
                </div>
            </form>
        </div>
    </div>