<script>
	import {db} from '$lib/firebase/client.js';
	import {userStore} from '$lib/store.js'
	import { collection, onSnapshot, query, where, orderBy } from 'firebase/firestore';
	import { jsPDF } from "jspdf";
    import autoTable from 'jspdf-autotable'
	import { onMount, afterUpdate } from 'svelte';
	
	let loans = []
	let currentPage = 1;
    let itemsPerPage = 12;

    let totalItems = 0;
    let totalPages = 0;
    let displayedItems = [];
	
	async function userLoans() {
        loans = []

        const q = query(collection(db, 'loanprocess'), where("owner", "==", $userStore.uid), orderBy("numberOfLoan", "desc") );
        const unsubscribe = onSnapshot(q, (querySnapshot) => {
            loans = querySnapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
			totalItems = loans.length;
            totalPages = Math.ceil(totalItems / itemsPerPage);
            updateDisplayedItems();
        });
        return () => unsubscribe();
        
    }

	function updateDisplayedItems() {
        let startIndex = (currentPage - 1) * itemsPerPage;
        let endIndex = startIndex + itemsPerPage;
        displayedItems = loans.slice(startIndex, endIndex);
    }

    function goToPage(pageNumber) {
        currentPage = pageNumber;
        updateDisplayedItems();
    }

    onMount(() => {
        userLoans();
    });

    afterUpdate(() => {
        updateDisplayedItems();
    });

	async function generateLoans(){
	 const paymentHistory = new jsPDF();

	 		paymentHistory.text('JEMPOWERS CREDIT COPORATION', 56, 22);
            paymentHistory.text('Loan History', 84, 37);
            // paymentHistory.text(startOfDay, 89, 45);
            paymentHistory.setFontSize(11);
            paymentHistory.text('San Pedro City, Laguna', 83, 27);
            paymentHistory.autoTable({
                theme: 'grid',
                margin: {top: 50, left:3}, 
                tableWidth: 205,
                headStyles: {fontSize: 6},
                style:{fontSize: 5, item:'center'},
                html: '#loan_history',
            });
            window.open(paymentHistory.output('bloburl'));

	}
  
</script>
<svelte:head>
	<title>JEM | Loans</title>
</svelte:head>

<section class="flex flex-col w-full h-screen overflow-y-auto">
	<div class="flex bg-white mb-3 w-full rounded-lg p-2">
		<span class=" flex-1 text-2xl max-sm:text-lg font-semibold">Loans</span>
	   <div class="flex flex-1 w-full items-center justify-end ">
		   <button class="btn btn-sm max-sm:btn-xs flex text-xs" on:click={generateLoans}>Generate PDF</button>
	   </div>
   </div>
		<div class="bg-white rounded-lg p-2 h-screen overflow-x-auto ">
			<table class="table max-sm:table-compact w-full" id="loan_history">
				<thead>
					<tr>
						<td class="px-6">#</td>
						<th scope="col" class="px-6">LOAN REF</th>
						<th scope="col" class="px-6">RELEASED DATE</th> 
						<th scope="col" class="px-6">DUE DATE</th> 
						<th scope="col" class="px-6">LOAN AMOUNT</th> 
						<th scope="col" class="px-6">D.PAYMENT</th> 
						<th scope="col" class="px-6">BALANCE</th> 
						<th scope="col" class="px-6">T.PAYMENT</th>
						<th scope="col" class="px-6">STATUS</th> 
					</tr>
				</thead>
				{#each displayedItems as loan}
					<tr>
						<td class="px-6">
							<p>{loan.numberOfLoan}</p>
						</td>
						<td class="px-6">
							{loan.loanNumber}
						</td>
						<td class="px-6">
							{loan.releaseDate}    
						</td>
						<td class="px-6">
							{loan.formattedDueDate}
						</td>
						<td class="px-6">
							{loan.loanAmount}        
						</td>
						<td class="px-6">
							{loan.dailyPayment}
						</td>
						<td class="px-6">
							{loan.balance}
						</td>
						<td class="px-6">
							{loan.totalPayment}
						</td>
						<td class="px-6">
							{loan.status}
						</td>
					</tr>
				{/each}
			</table>	
		</div>

		<!-- Pagination -->
<div class="mt-4 flex justify-center">
    <button class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200" disabled={currentPage === 1} on:click={() => goToPage(1)}>
      First
    </button>
    <button class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200" disabled={currentPage === 1} on:click={() => goToPage(currentPage - 1)}>
      Previous
    </button>

    <!-- {#if currentPage > 3}
      <span class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200">...</span>
    {/if} -->

    <!-- {#each Array.from({ length: totalPages }, (_, i) => i + 1) as page}
      {#if currentPage - 2 <= page && page <= currentPage + 2}
        <button class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200" aria-current={currentPage === page} on:click={() => goToPage(page)}>
          {page}
        </button>
      {/if}
    {/each} -->

    <!-- {#if currentPage < totalPages - 2}
      <span class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200">...</span>
    {/if} -->

    <button class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200" disabled={currentPage === totalPages} on:click={() => goToPage(currentPage + 1)}>
      Next
    </button>
    <button class="cursor-pointer px-2 py-1 rounded hover:bg-gray-200" disabled={currentPage === totalPages} on:click={() => goToPage(totalPages)}>
      Last
    </button>
  </div>
</section>	

