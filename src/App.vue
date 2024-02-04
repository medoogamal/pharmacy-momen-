<template>
  <header class="bg-blue-500 p-4" :class="search ? 'fixed w-full' : ''">
    <div class="container mx-auto flex items-center justify-between">
    <div class="flex items-center justify-between">
      <div class="flex items-center space-x-4" v-if="!search">
        <img
          src="/247-2471138_pharmacy-symbol-hd-png-download.png"
          alt="Pharmacy Logo"
          class="w-12 h-12 rounded-full"
        />
        <h1 class="text-white text-2xl font-semibold">
          M Pharmacy صيدليه د/مؤمن لطفي
        </h1>
      </div>
      <div class="flex items-center gap-2 ml-[550px]">
        <h1 class="text-white text-lg font-bold">Kitchen Girl</h1>
        <img src="/nada.jpg" class="w-[60px]  rounded-full" alt="">
        <img src="/r.jpg" class="w-[60px]  rounded-full" alt="">
      </div>
    </div>
      

      <div
        @click="openSearch"
        class="fixed bottom-20 text-white p-[10px] rounded-xl bg-green-500 text-xl font-semibold"
      >
        بحث
      </div>
      <div class="flex items-center space-x-4" v-if="search">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search"
          class="px-4 py-2 rounded-lg" 
        />
        <div
          @click="closeSearch"
          class="text-white p-2 rounded-xl bg-red-500 text-xl font-semibold"
        >
          اغلاق
        </div>
      </div>
    </div>
  </header>

  <div :class="search ? 'pt-[21%]' : ''">
    <!-- Display drugs from local storage -->
    <div
  v-for="(drug, index) in filteredDrugs"
  :key="index"
  :class="{ 'bg-gray-500': drug.bookmarked }"  
  class="flex items-center justify-between border p-2"
>
  <label :for="'drug' + index" class="block capitalize">{{ drug.name }}</label>
  <div class="flex items-center gap-3">
    <!-- Add bookmark button with click event -->
    <button
      @click="toggleBookmark(index)"
      class="bg-yellow-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
    >
      {{ drug.bookmarked ? 'Remove Bookmark' : 'Add Bookmark' }}
    </button>
    <!-- Add edit button and call editDrug method -->
    <button
      @click="editDrug(index)"
      class="bg-blue-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl md:block hidden"
    >
      Edit
    </button>
    <button
      @click="deleteDrug(index)"
      class="bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl md:block hidden"
    >
      Delete
    </button>
    <button
      @click="decrementValue(index)"
      class="bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
    >
      -
    </button>
    <input
      :type="drug.type"
      :name="'drug' + index"
      :id="'drug' + index"
      :min="0"
      v-model="drug.value"
      class="border border-black w-[70px] rounded py-1 px-3 block"
    />
    <button
      @click="incrementValue(index)"
      class="bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
    >
      +
    </button>
  </div>
</div>

    <div
      class="flex flex-col sm:flex-row items-center justify-between border p-2"
    >
      <input
        type="text"
        v-model="newDrugName"
        @keyup.enter="addNewDrug"
        placeholder="Enter Drug Name"
        class="border border-black w-full sm:w-[200px] rounded py-1 px-3 mb-2 sm:mb-0"
      />
      <div class="flex items-center gap-3">
        <button
    @click="toggleNewDrugBookmark"
    class="bg-yellow-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
  >
    {{ newDrugBookmarked ? 'Remove Bookmark' : 'Add Bookmark' }}
  </button>
        <button
          @click="decrementNewDrugCount"
          class="bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
        >
          -
        </button>
        <input
          type="number"
          v-model="newDrugCount"
          :min="1"
          placeholder="Enter Count"
          @keyup.enter="addNewDrug"
          class="border border-black w-full sm:w-[70px] rounded py-1 px-3"
        />
        <button
          @click="incrementNewDrugCount"
          class="bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl"
        >
          +
        </button>
      </div>
      <button
        @click="addNewDrug"
        class="bg-green-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl mt-2 sm:mt-0"
      >
        Add Drug
      </button>
    </div>
<div class="flex items-center justify-between md:px-5 px-1">
    <button
      @click="generatePDF"
      class=" bg-gray-300 py-2 px-4 rounded-xl hover:bg-gray-500 font-extrabold"
    >
      Generate PDF
    </button>
    <button @click="clearLocalStorage" class=" bg-red-500 py-2 px-3 rounded-lg text-white font-extrabold text-xl mt-2">
       Delete All
    </button>
  </div>
    <h1 class="p-3"><b>Made By:</b> Mohamed Gamal</h1>
  </div>
</template>

<script>
import jsPDF from "jspdf";
import "jspdf-autotable";

const LOCAL_STORAGE_KEY = "pharmacyDrugs";

export default {
  data() {
    return {
      drugs: [], // Initialize drugs as an empty array
      newDrugName: "",
      newDrugCount: 1,
      newDrugBookmarked: false,
      search: false,
      searchQuery: "",
    };
  },
  computed: {
    filteredDrugs() {
      return this.drugs.filter((drug) => {
        return drug.name.toLowerCase().includes(this.searchQuery.toLowerCase());
      });
    },
  },
  methods: {
    generatePDF() {
      const pdf = new jsPDF();

      // Filter drugs with a value greater than 0
      const filteredDrugs = this.drugs.filter((drug) => drug.value > 0);
      pdf.text("M Pharmacy | Dr.Momen Lotfy", 14, 10);

      // Define the table headers
      const headers = [["Drug Name", "Value"]];

      // Extract data for the table from the filtered drugs
      const data = filteredDrugs.map((drug) => [drug.name, drug.value]);

      // Create the table with jsPDF-AutoTable
      pdf.autoTable({
        head: headers,
        body: data,
        startY: 15,
        // Initial y-coordinate for the table
      });
      const date = new Date();
      const options = {
        weekday: "short",
        year: "numeric",
        month: "short",
        day: "numeric",
      };
      const formattedDate = date.toLocaleDateString(undefined, options);
      pdf.save(`${formattedDate}.pdf`);
    },
    openSearch() {
      this.search = true;
    },
    closeSearch() {
      this.search = false;
      this.searchQuery = "";
    },
    decrementValue(index) {
      if (this.drugs[index].value > 0) {
        this.drugs[index].value--;
      }
    },
    toggleNewDrugBookmark() {
      // Toggle the bookmark status for the newly added drug
      this.newDrugBookmarked = !this.newDrugBookmarked;
    },
    incrementValue(index) {
      this.drugs[index].value++;
    },
    decrementNewDrugCount() {
      if (this.newDrugCount > 0) {
        this.newDrugCount--;
      }
    },
    incrementNewDrugCount() {
      this.newDrugCount++;
    },
    toggleBookmark(index) {
      // Toggle the 'bookmarked' property for the selected drug
      this.drugs[index].bookmarked = !this.drugs[index].bookmarked;
      // Save the updated drugs array to local storage
      this.saveDrugsToLocalStorage();
    },
    editDrug(index) {
  // Get the drug to be edited
  const drugToEdit = this.drugs[index];

  // Prompt the user for the new drug name
  const newDrugName = window.prompt("ادخل اسم الدواء الجديد:", drugToEdit.name);

  if (newDrugName !== null) {
    // Remove the drug from its current position in the array
    this.drugs.splice(index, 1);

    // Update the drug name
    drugToEdit.name = newDrugName;

    // Find the correct index to insert the edited drug in alphabetical order
    const insertIndex = this.drugs.findIndex(
      (drug) => drug.name.localeCompare(newDrugName) > 0
    );

    // If insertIndex is -1, it means the edited drug should be added at the end
    if (insertIndex === -1) {
      this.drugs.push(drugToEdit);
    } else {
      // Insert the edited drug at the correct index
      this.drugs.splice(insertIndex, 0, drugToEdit);
    }

    // Save the updated drugs array to local storage
    this.saveDrugsToLocalStorage();
  }
},

    deleteDrug(index) {
      const confirmDelete = window.confirm("هل أنت متأكد من حذف هذا الدواء؟");

      if (confirmDelete) {
        // Remove the drug at the specified index
        this.drugs.splice(index, 1);
        // Save the updated drugs array to local storage
        this.saveDrugsToLocalStorage();
      }
    },

    loadDrugsFromLocalStorage() {
      const storedDrugs = localStorage.getItem(LOCAL_STORAGE_KEY);

      if (storedDrugs) {
        try {
          // Parse the stored JSON data and assign it to the 'drugs' data property
          this.drugs = JSON.parse(storedDrugs);
        } catch (error) {
          console.error("Error parsing drugs from local storage:", error);
        }
      }
    },
    clearLocalStorage() {
    // Show a confirmation dialog before clearing local storage
    const confirmDelete = window.confirm("هل انت متاكد من انك تريد حئف كل الادويه الحالية");
    
    if (confirmDelete) {
      // Clear all items in local storage
      localStorage.removeItem(LOCAL_STORAGE_KEY);
      // Clear the drugs array in your component
      this.drugs = [];
    }
  },

    saveDrugsToLocalStorage() {
      try {
        // Stringify the 'drugs' array and save it to local storage
        localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(this.drugs));
      } catch (error) {
        console.error("Error saving drugs to local storage:", error);
      }
    },

  addNewDrug() {
    if (this.newDrugName && this.newDrugCount >= 0) {
        // Create a new drug object with bookmark status
        const newDrug = {
          name: this.newDrugName,
          type: "text",
          value: this.newDrugCount,
          bookmarked: this.newDrugBookmarked,
        };

    // Find the correct index to insert the new drug in alphabetical order
    const insertIndex = this.drugs.findIndex(
      (drug) => drug.name.localeCompare(newDrug.name) > 0
    );

    // If insertIndex is -1, it means the new drug should be added at the end
    if (insertIndex === -1) {
      this.drugs.push(newDrug);
    } else {
      // Insert the new drug at the correct index
      this.drugs.splice(insertIndex, 0, newDrug);
    }

    // Clear the input fields after adding a new drug
    this.newDrugName = "";
    this.newDrugCount = 1;
    this.saveDrugsToLocalStorage();
  }
},
 // Use the created lifecycle hook to load drugs from local storage when the component is created
 
  },
   created() {
    // Use the created lifecycle hook to load drugs from local storage when the component is created
    this.loadDrugsFromLocalStorage();
  },
  
};
</script>



