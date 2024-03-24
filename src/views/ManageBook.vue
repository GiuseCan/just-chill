<template>
  <div class="text-gray-900 bg-gray-200">
    <div class="p-4 flex">
      <h1 class="text-3xl font-bold text-gray-600">Đặt lịch</h1>
    </div>
    <div class="px-3 py-4 flex justify-center">
      <table class="w-full text-md bg-white shadow-md rounded mb-4">
        <tbody>
          <tr class="border-b">
            <th class="text-left p-3 px-5">ID vé</th>
            <th class="text-left p-3 px-5">Tên người đặt</th>
            <th class="text-left p-3 px-5">Tổng chi phí (VND)</th>
            <th class="text-left p-3 px-5">Thanh toán</th>
            <th></th>
          </tr>
          <tr
            v-for="(booking, index) in displayedBooking"
            :key="index"
            class="border-b hover:bg-orange-100 bg-gray-100"
          >
            <td class="p-3 px-5">
              <input
                v-model="booking.id_ticket"
                type="text"
                disabled="true"
                class="bg-transparent border-none"
              />
            </td>
            <td class="p-3 px-5">
              <input
                :value="setUserName(booking)"
                type="text"
                disabled="true"
                class="bg-transparent border-none"
              />
            </td>
            <td class="p-3 px-5">
              <input
                v-model="formattedTotalAmounts[index]"
                type="text"
                disabled="true"
                class="bg-transparent border-none"
              />
            </td>
            <td class="p-3 px-5">
              <!-- v-model="booking.total_amount" -->
              <input
                :value="
                  booking.paid === 1 ? 'Đã thanh toán' : 'Chưa thanh toán'
                "
                type="text"
                disabled="true"
                class="bg-transparent border-none"
              />
            </td>
            <td class="p-3 px-5 flex justify-end">
              <button
                class="mr-3 text-sm bg-blue-500 hover:bg-blue-700 text-white py-1 px-2 rounded focus:outline-none focus:shadow-outline"
                @click="editBooking(booking)"
              >
                View
              </button>
              <button
                type="button"
                class="text-sm bg-red-500 hover:bg-red-700 text-white py-1 px-2 rounded focus:outline-none focus:shadow-outline"
                @click="deleteBooking(booking)"
              >
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- Pagination -->
    <div class="flex justify-center">
      <button
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-l"
        :disabled="currentPage === 1"
        @click="goToPage(currentPage - 1)"
      >
        Previous
      </button>
      <template v-if="totalPages <= 5">
        <!-- Hiển thị tất cả các trang nếu tổng số trang không vượt quá 5 -->
        <button
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
          v-for="pageNumber in totalPages"
          :key="pageNumber"
          :class="{ 'bg-blue-700': currentPage === pageNumber }"
          @click="goToPage(pageNumber)"
        >
          {{ pageNumber }}
        </button>
      </template>
      <template v-else>
        <span class="px-4">...</span>
        <!-- Hiển thị trang hiện tại và 2 trang liền kề nếu có -->
        <template v-if="currentPage <= 3">
          <!-- Nếu trang hiện tại là 1, 2 hoặc 3 -->
          <button
            v-for="pageNumber in 3"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
            :class="{ 'bg-blue-700': currentPage === pageNumber }"
            :key="pageNumber"
            @click="goToPage(pageNumber)"
          >
            {{ pageNumber }}
          </button>
        </template>
        <template v-else-if="currentPage >= totalPages - 2">
          <!-- Nếu trang hiện tại là ở 3 trang cuối cùng -->
          <button
            v-for="pageNumber in [totalPages - 2, totalPages - 1, totalPages]"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
            :class="{ 'bg-blue-700': currentPage === pageNumber }"
            :key="pageNumber"
            @click="goToPage(pageNumber)"
          >
            {{ pageNumber }}
          </button>
        </template>
        <template v-else>
          <!-- Trang hiện tại ở giữa -->
          <button
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
            @click="goToPage(currentPage - 1)"
          >
            {{ currentPage - 1 }}
          </button>
          <button
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
            :class="{ 'bg-blue-700': currentPage === currentPage }"
            @click="goToPage(currentPage)"
          >
            {{ currentPage }}
          </button>
          <button
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4"
            @click="goToPage(currentPage + 1)"
          >
            {{ currentPage + 1 }}
          </button>
        </template>
        <span class="px-4">...</span>
      </template>
      <button
        class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-r"
        :disabled="currentPage === totalPages"
        @click="goToPage(currentPage + 1)"
      >
        Next
      </button>
    </div>

    <!-- Model Edit -->
    <div
      v-if="showModal"
      class="fixed inset-0 z-50 flex items-center justify-center overflow-auto bg-black bg-opacity-50"
      @click.self="closeModal"
    >
      <div class="bg-white p-8 rounded-lg shadow-lg max-w-md">
        <!-- Nội dung của modal -->
        <h2 class="text-xl font-semibold mb-4">Thông tin đặt lịch</h2>
        <form
          @submit.prevent="update"
          class="grid grid-cols-1 gap-6 mt-8 md:grid-cols-2"
        >
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Tên người đặt</label
            >
            <!-- v-model="currentBooking.user_name !== '' ? currentBooking.customer_name : currentBooking.user_name" -->
            <input
              :value="setUserName(currentBooking)"
              disabled="true"
              type="text"
              placeholder="Nguyen Cong Can"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>

          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Email người dùng</label
            >
            <input
              disabled="true"
              :value="setEmail(currentBooking)"
              type="text"
              placeholder="canbl2002@gmail.com"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Số điện thoại</label
            >
            <input
              disabled="true"
              :value="setPhone(currentBooking)"
              type="text"
              placeholder="09xxxxxxxx"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Tổng chi phí (VND)</label
            >
            <input
              disabled="true"
              :value="formatCurrencyVND(currentBooking.total_amount)"
              type="text"
              placeholder="VND"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Ngày xuất phát</label
            >
            <input
              disabled="true"
              v-model="currentBooking.time_start"
              type="date"
              placeholder="10/10/2025"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Ngày về</label
            >
            <input
              disabled="true"
              v-model="currentBooking.time_end"
              type="date"
              placeholder="20/10/2024"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Điểm khởi hành</label
            >
            <select
              disabled="true"
              v-model="currentBooking.departure_id"
              class="w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            >
              <option
                v-for="departure in departures"
                :key="departure.location_id"
                :value="departure.location_id"
              >
                {{ departure.location_name }}
              </option>
            </select>
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Điểm đến</label
            >
            <select
              disabled="true"
              v-model="currentBooking.destination_id"
              class="w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            >
              <option
                v-for="destination in destinations"
                :key="destination.location_id"
                :value="destination.location_id"
              >
                {{ destination.location_name }}
              </option>
            </select>
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Các địa điểm tham quan</label
            >
            <ul
              class="block h-20 overflow-y-auto w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            >
              <li
                v-for="(visit, index) in currentBooking.visit_list"
                :key="index"
              >
                {{ index + 1 }}: {{ visit }}
              </li>
            </ul>
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Trạng thái thanh toán</label
            >
            <select
              disabled="true"
              v-model="currentBooking.paid"
              class="w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            >
              <option value="1">Đã thanh toán</option>
              <option value="0">Chưa thanh toán</option>
            </select>
          </div>

          <!-- <button
            type="submit"
            @click="update()"
            class="flex items-center justify-between w-full px-6 py-3 text-sm tracking-wide text-white capitalize transition-colors duration-300 transform bg-blue-500 rounded-md hover:bg-blue-400 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-50"
          >
            <span>Lưu chỉnh sửa</span>

            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-5 h-5 rtl:-scale-x-100"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                fill-rule="evenodd"
                d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                clip-rule="evenodd"
              />
            </svg>
          </button> -->
        </form>

        <!-- Nút để đóng modal -->
        <button
          class="mt-4 text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 py-1 px-2 rounded focus:outline-none focus:shadow-outline"
          @click="closeModal"
        >
          Close
        </button>
      </div>
    </div>

    <!-- Modal Del -->
    <div
      v-if="showModalDel"
      class="fixed inset-0 z-50 flex items-center justify-center overflow-auto bg-black bg-opacity-50"
      @click.self="closeModalDel"
    >
      <div class="bg-white p-8 rounded-lg shadow-lg max-w-md">
        <!-- Nội dung của modal -->
        <h2 class="text-xl font-semibold mb-4">Xác nhận xoá đặt lịch</h2>
        <p>Bạn muốn xoá đặt lịch?</p>
        <p>ID vé: {{ currentBooking.id_ticket }}</p>
        <p>Tên người đặt: {{ currentBooking.user_name }}</p>
        <p>Chi phí: {{ formattedTotalAmount }} VND</p>
        <!-- Nút để xác nhận xoá -->
        <button
          class="mt-4 text-sm bg-red-500 hover:bg-red-700 text-white py-1 px-2 rounded focus:outline-none focus:shadow-outline mr-2"
          @click="deleteItem"
        >
          Xoá
        </button>
        <!-- Nút để trở lại -->
        <button
          class="mt-4 text-sm bg-gray-300 hover:bg-gray-400 text-gray-800 py-1 px-2 rounded focus:outline-none focus:shadow-outline"
          @click="closeModalDel"
        >
          Trở về
        </button>
      </div>
    </div>

    <!-- Modal Add -->
    <div
      v-if="showModalAdd"
      class="fixed inset-0 z-50 flex items-center justify-center overflow-auto bg-black bg-opacity-50"
      @click.self="closeModalAdd"
    >
      <div class="bg-white p-8 rounded-lg shadow-lg max-w-md">
        <!-- Nội dung của modal -->
        <h2 class="text-xl font-semibold mb-4">Đặt lịch</h2>
        <form
          @submit.prevent="update"
          class="grid grid-cols-1 gap-6 mt-8 md:grid-cols-2"
        >
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >Tên địa điểm</label
            >
            <input
              v-model="user_name"
              type="text"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <div>
            <label class="block mb-2 text-sm text-gray-600 dark:text-gray-200"
              >???</label
            >
            <input
              v-model="user_name"
              type="text"
              class="block w-full px-5 py-3 mt-2 text-gray-700 placeholder-gray-400 bg-white border border-gray-200 rounded-md dark:placeholder-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:border-gray-700 focus:border-blue-400 dark:focus:border-blue-400 focus:ring-blue-400 focus:outline-none focus:ring focus:ring-opacity-40"
            />
          </div>
          <button
            type="submit"
            @click="update()"
            class="flex items-center justify-between w-full px-6 py-3 text-sm tracking-wide text-white capitalize transition-colors duration-300 transform bg-blue-500 rounded-md hover:bg-blue-400 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-50"
          >
            <span>Đặt lịch</span>

            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-5 h-5 rtl:-scale-x-100"
              viewBox="0 0 20 20"
              fill="currentColor"
            >
              <path
                fill-rule="evenodd"
                d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                clip-rule="evenodd"
              />
            </svg>
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, watch, computed } from "vue";
import axios from "axios";

const bookings = ref([]);
const currentBooking = ref({});
const destinations = ref([]);
const departures = ref([]);

onMounted(async () => {
  try {
    const booking_list = await axios.get(
      "http://localhost:8080/smart_travel_api/admin/api/manageBooking/showAllBooking.php"
    );
    bookings.value = booking_list.data;
    console.log(bookings.value);
    // debugger;
  } catch (error) {
    console.error(error);
  }
  try {
    const destinationList = await axios.get(
      "http://localhost:8080/smart_travel_api/admin/api/manageVisit/showAllDestination.php"
    );
    destinations.value = destinationList.data;
    console.log(destinations.value);
  } catch (error) {
    console.error(error);
  }

  try {
    const departureList = await axios.get(
      "http://localhost:8080/smart_travel_api/admin/api/manageVisit/showAllDeparture.php"
    );
    departures.value = departureList.data;
    console.log(departures.value);
  } catch (error) {
    console.error(error);
  }
  formatData();
});

// Số lượng mục trên mỗi trang
const itemsPerPage = ref(5);

// Trang hiện tại
const currentPage = ref(1);

// Tính tổng số trang dựa trên số lượng mục và số lượng mục trên mỗi trang
const totalPages = computed(() =>
  Math.ceil(bookings.value.length / itemsPerPage.value)
);

// Tính chỉ mục bắt đầu của mục đầu tiên trên trang hiện tại
const startIndex = computed(() => (currentPage.value - 1) * itemsPerPage.value);

// Lọc và hiển thị danh sách người dùng trên trang hiện tại
const displayedBooking = computed(() =>
  bookings.value.slice(startIndex.value, startIndex.value + itemsPerPage.value)
);

// Chuyển đến trang cụ thể
const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
  }
};

function setUserName(booking) {
  return booking.user_name ? booking.user_name : booking.customer_name;
}
function setEmail(booking) {
  return booking.email ? booking.email : booking.email_customer;
}
function setPhone(booking) {
  return booking.phone_number
    ? booking.phone_number
    : booking.phone_number_customer;
}
// Phương thức để chỉnh sửa người dùng
const editBooking = (booking) => {
  console.log(booking);
  currentBooking.value = { ...booking };
  showModal.value = true; // Hiển thị modal
};
const deleteBooking = (booking) => {
  currentBooking.value = { ...booking };
  showModalDel.value = true; // Hiển thị modal
};
// console.log(currentBooking.value);
const formattedTotalAmount = computed(() => {
  // Lấy giá trị của tuition_fee từ currentVisit
  const totalAmount = currentBooking.value.total_amount;
  // Định dạng số theo kiểu tiền VND (tách 3 chữ số 1 bằng dấu chấm từ phải sang trái)
  return totalAmount.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
});

function formatCurrencyVND(number) {
  // Chuyển số thành chuỗi và đảm bảo là kiểu số nguyên
  number = parseInt(number).toString();

  // Tạo biến lưu trữ chuỗi kết quả
  let result = "";

  // Duyệt qua chuỗi số từ phải sang trái
  for (let i = number.length - 1, j = 0; i >= 0; i--, j++) {
    // Nếu đến vị trí thứ 3 từ phải sang trái và không phải vị trí cuối cùng
    if (j % 3 === 0 && j !== 0) {
      // Thêm dấu chấm vào chuỗi kết quả
      result = "." + result;
    }
    // Thêm ký tự hiện tại vào chuỗi kết quả
    result = number[i] + result;
  }

  // Trả về chuỗi kết quả đã được định dạng
  return result;
}
// Sử dụng một biến ref để theo dõi trạng thái của modal
const showModal = ref(false);

// Hàm để đóng modal
const closeModal = () => {
  showModal.value = false;
};

// Sử dụng một biến ref để theo dõi trạng thái của modal
const showModalDel = ref(false);

// Hàm để đóng modal
const closeModalDel = () => {
  showModalDel.value = false;
};

// Hàm để xác nhận xoá
async function deleteItem() {
  try {
    const response = await axios.delete(
      `http://localhost:8080/smart_travel_api/admin/api/manageBooking/deleteBooking.php?id=${currentBooking.value.id_booking}`
    );
    alert("Xoá thành công");
    closeModalDel();

    const updatedBookingIndex = displayedBooking.value.findIndex(
      (booking) => booking.id === currentBooking.value.id
    );
    if (updatedBookingIndex !== -1) {
      displayedBooking.value.splice(updatedBookingIndex, 1); // Xoá user khỏi displayedUsers
    }
  } catch (error) {
    console.error("Error:", error);
    alert("Xoá thất bại. Xin vui lòng thử lại!");
  }
}

const showModalAdd = ref(false);

// Hàm để đóng modal
const closeModalAdd = () => {
  showModalAdd.value = false;
};

// Hàm để thêm người dùng (ở đây bạn có thể thêm logic xử lý thêm người dùng)
const addUser = () => {
  // Đóng modal sau khi thêm người dùng
  closeModalAdd();
};
const formattedTotalAmounts = ref([]);
function formatData() {
  formattedTotalAmounts.value = bookings.value.map((booking) =>
    formatCurrency(booking.total_amount)
  );
}
function formatCurrency(total) {
  // Chuyển đổi số thành chuỗi và tách thành mảng các ký tự
  const strTotal = total.toString().split("");
  // Tạo mảng mới để lưu trữ chuỗi kết quả
  let result = [];
  // Biến đếm số ký tự
  let count = 0;

  // Lặp qua từng ký tự trong chuỗi
  for (let i = strTotal.length - 1; i >= 0; i--) {
    // Thêm ký tự vào đầu mảng kết quả
    result.unshift(strTotal[i]);
    // Tăng biến đếm số ký tự
    count++;
    // Nếu đã thêm 3 ký tự và chưa phải là ký tự cuối cùng
    if (count % 3 === 0 && i !== 0) {
      // Thêm dấu chấm vào đầu mảng kết quả
      result.unshift(".");
    }
  }

  // Ghép mảng kết quả thành chuỗi và thêm ký tự VND vào cuối
  return result.join("") + " VND";
}
</script>

<style scoped></style>
