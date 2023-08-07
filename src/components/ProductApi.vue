<template>
  <div>
    <h1 class="display-2">Product</h1>

    <!-- Search input for filtering -->
    <input
      type="text"
      v-model="searchQuery"
      placeholder="Search by Product"
      style="
        color: black;
        align-items: start;
        border-radius: 5px;
        padding: 0 40px;
        margin: 20px;
        border: 1px solid rgb(84, 88, 90);
        background-color: rgb(242, 236, 236);
      "
    />

    <!-- Dropdown to select rows per page -->

    <table class="table table-striped table-bordered">
      <!-- Table headers -->
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Item Number</th>
          <th>Cost price</th>
          <th>Unit price</th>

          <th>Print</th>
        </tr>
      </thead>

      <!-- Table body with paginated and filtered items -->
      <tbody>
        <tr v-for="item in paginatedItems" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.laoname }}</td>
          <td>{{ item.itemnumber }}</td>

          <!-- <td>{{ item.costprice + " ກີບ" }}</td> -->
          <td>
            {{ (parseFloat(item.costprice) / 1000).toFixed(3) + " ກີບ" }}
          </td>
          <td>
            {{ (parseFloat(item.unitprice) / 1000).toFixed(3) + " ກີບ" }}
          </td>

          <td>
            <button
              class="btn btn-success"
              @click="
                generateBarcode(
                  item.itemnumber,
                  (parseFloat(item.unitprice) / 1000).toFixed(3)
                )
              "
            >
              <i class="fas fa-print"></i>
              Print
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Pagination controls -->
    <div class="pagination">
      <button
        class="btn btn-primary"
        @click="prevPage"
        :disabled="currentPage === 1"
      >
        Previous
      </button>
      <span class="page-number">Page {{ currentPage }}/{{ totalPages }}</span>
      <button
        class="btn btn-dark"
        @click="nextPage"
        :disabled="currentPage === totalPages"
      >
        Next
      </button>
    </div>
  </div>
</template>
<style>
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.page-number {
  font-size: 18px;
  margin-right: 20px;
}

.btn {
  margin: 5px;
}
</style>

<script>
import axios from "axios";
import "bootstrap/dist/css/bootstrap.min.css"; // Import Bootstrap CSS
import "bootstrap"; // Import Bootstrap JavaScript
import JsBarcode from "jsbarcode";
import { apiUrl } from "./apiConfig"; // Import the apiUrl constant

export default {
  // Vue component options
  data() {
    return {
      items: [], // Initialize an empty array to hold the fetched items
      itemsPerPage: 10, // Number of items to display per page
      currentPage: 1, // Current page number
      searchQuery: "", // Holds the user's search query
      count: 3, // Number of items to display
    };
  },

  mounted() {
    // Fetch data when the component is mounted
    this.fetchData();
  },

  computed: {
    // Calculate the total number of pages based on the number of items and itemsPerPage
    totalPages() {
      return Math.ceil(this.filteredItems.length / this.itemsPerPage);
    },
    // Slice the items array to get the items for the current page
    paginatedItems() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      return this.filteredItems.slice(startIndex, endIndex);
    },
    // Filter the items based on the search query
    filteredItems() {
      const query = this.searchQuery.trim().toLowerCase();
      if (!query) {
        return this.items; // If no search query, return all items
      } else {
        return this.items.filter((item) =>
          item.laoname.toLowerCase().includes(query)
        );
      }
    },
  },

  methods: {
    fetchData() {
      const apiUrl = "http://172.22.7.22:8888/api/products/1001";

      // Make the GET API call using Axios
      axios
        .get(apiUrl)
        .then((response) => {
          // Handle the API response
          this.items = response.data; // Update the items array with the fetched data
        })
        .catch((error) => {
          // Handle any error that occurred during the API call
          console.error("Error fetching data:", error);
        });
    },

    // fetchData() {
    //   const apiUrl = process.env.VUE_APP_API_URL;

    //   axios
    //     .get(apiUrl)
    //     .then((response) => {
    //       this.items = response.data;
    //     })
    //     .catch((error) => {
    //       console.error("Error fetching data:", error);
    //     });
    // },

    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },

    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },

    generateBarcode(account, productName) {
      // Create a new window to show the barcode and timestamp
      const maxProductNameLength = 26;
      const newWindow = window.open("", "_blank");

      if (productName.length > maxProductNameLength) {
        productName = productName.substring(0, maxProductNameLength) + "...";
      }
      // Generate the barcode using JsBarcode library and set it as the content of the new window
      if (newWindow) {
        newWindow.document.write("<html><body >");

        // Generate three barcodes with names for the same product in three columns
        newWindow.document.write(
          "<div style=' width: 150mm; display: flex; justify-content: space-between;'>"
        );
        for (let i = 0; i < this.count; i++) {
          //size space between barcode
          newWindow.document.write(
            "<div style='width: 33.33%;margin:-20px 20px 0px 20px '>"
          );
          newWindow.document.write(
            `<p style="font-family: 'Noto Sans Lao', sans-serif;,width:150px;text-align: center; font-size: 12; margin-top: 10px;margin-bottom:-9px;padding:5px 0 ">${productName}</p>`
          );
          newWindow.document.write(
            `<svg id="barcodeElement${i}" style="text-align: center;"></svg>`
          );
          newWindow.document.write("<style>");
          newWindow.document.write(`
    svg {
      display: block;
      margin: 0 auto;
      width: 100%;
    }
  `);
          newWindow.document.write("</style>");

          newWindow.document.write("</div>");
        }
        newWindow.document.write("</div>");

        newWindow.document.write("</body></html>");

        // Generate the barcodes using JsBarcode library and append them to the new window
        for (let i = 0; i < this.count; i++) {
          JsBarcode(
            newWindow.document.getElementById(`barcodeElement${i}`),
            account,
            {
              format: "CODE128",
              width: 1.7,
              height: 65,
              displayValue: true,
              fontSize: 20,
              margin: 0,
              background: "#ffffff",
              lineColor: "#000000",
              fontOptions: "bolder",
            }
          );
        }

        newWindow.print();
      } else {
        alert("Popup blocked! Please allow popups to see the barcode.");
      }
    },
  },
};
</script>
