# namco-backoffice-source

```ts
import {
  VNPAY_GATEWAY_URL,
  VNPAY_TMN_CODE,
  VNPAY_SECRET,
  BASE_URL,
} from "$env/static/private";
import { VNPay } from "vnpay";

export default async function initVnpay() {
  return new VNPay({
    paymentGateway: VNPAY_GATEWAY_URL,
    tmnCode: VNPAY_TMN_CODE,
    secureSecret: VNPAY_SECRET,
    returnUrl: BASE_URL + "/book/confirm",
  });
}
```

```ts
import { WHATSAPP_API_URL } from "$env/static/private";

const headers = {
  "content-type": "application/json",
};

type StandardResponse = Promise<{ status: string; message: string }>;

type Endpoint = "create-group" | "add-guest-to-group" | "send-message-to-group";

async function callWhatsappAPi(endpoint: Endpoint, body: any) {
  const res = await fetch(WHATSAPP_API_URL + "/" + endpoint, {
    method: "POST",
    headers,
    body: JSON.stringify(body),
  });

  return await res.json();
}

async function createGroup(
  groupName: string,
  imageUrl: string,
  participants: string[]
): Promise<{
  status: string;
  message: string;
  groupId: string;
  inviteCode: string;
}> {
  return await callWhatsappAPi("create-group", {
    groupName,
    imageUrl,
    participants,
  });
}

async function addGuestToGroup(
  groupId: string,
  participants: string[]
): StandardResponse {
  return await callWhatsappAPi("add-guest-to-group", {
    groupId,
    participants,
  });
}

async function sendMessageToGroup(
  groupId: string,
  message: string
): StandardResponse {
  return await callWhatsappAPi("send-message-to-group", {
    groupId,
    message,
  });
}

async function alive(): StandardResponse {
  const res = await fetch(WHATSAPP_API_URL + "/alive");
  return await res.json();
}

const whatsapp = {
  createGroup,
  addGuestToGroup,
  sendMessageToGroup,
  alive,
};

export default whatsapp;
```

```html
<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>Thời gian</th>
        <th />
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>Kiểm tra vào</th>
        <td>18:00</td>
      </tr>
      <tr>
        <th>Bữa tối</th>
        <td>19:00</td>
      </tr>
      <tr>
        <th>Bữa sáng</th>
        <td>07:15</td>
      </tr>
      <tr>
        <th>kiểm tra giờ giấc</th>
        <td>09:00</td>
      </tr>
    </tbody>
  </table>
</div>

<img src="/img/bathroom.png" />
```

```html
<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>Cơm gia đình</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>Ba chỉ lơn rang</td>
      </tr>
      <tr>
        <td>Trứng rán</td>
      </tr>
      <tr>
        <td>Nem chay</td>
      </tr>
      <tr>
        <td>Khoai tây xào</td>
      </tr>
      <tr>
        <td>Đậu phụ sốt cà chua</td>
      </tr>
      <tr>
        <td>Rau theo mùa</td>
      </tr>
      <tr>
        <td>Cơm,cánh nóng</td>
      </tr>
      <tr>
        <td>Khoai tây chiên</td>
      </tr>
    </tbody>
  </table>
</div>

<img src="/img/meal.png" />
<img src="/img/condiments.png" />
<img src="/img/bathroom.png" />
```

```html
<script lang="ts">
  import type { Guest } from "$lib/types";
  export let guests: Guest[];
</script>

<div class="table">
  <table>
    <tbody>
      {#each guests as { guestName }, i}
      <tr>
        <td style="width: 2rem">{i}</td>
        <td style="text-transform: uppercase">{guestName}</td>
        <td style="width: 3rem" />
      </tr>
      {/each}
    </tbody>
  </table>
</div>
```

```html
<script lang="ts">
  import { locales, locale } from "$lib/localise.js";
</script>

<select bind:value="{$locale}">
  {#each locales as { isoLanguageCode, name, emoji }}
  <option value="{isoLanguageCode}">{name} {emoji}</option>
  {/each}
</select>
```

```html
<script lang="ts">
  import type { PurchaseOrderRow, Transaction } from "$lib/types";
  import t from "$lib/localise";
  import { currency } from "$lib/utils";

  export let purchaseOrder: PurchaseOrderRow[];
  export let transactions: Transaction[];

  const totalCost = purchaseOrder.reduce(
    (accumulator, item) => accumulator + item.unitPrice * item.quantity,
    0
  );

  const totalPaid = transactions.reduce(
    (accumulator, trans) => accumulator + trans.amount,
    0
  );
</script>

<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>{$t.itemName}</th>
        <th>{$t.unitType}</th>
        <th>{$t.unitPrice}</th>
        <th>{$t.quantity}</th>
        <th>{$t.totalAmount}</th>
      </tr>
    </thead>
    <tbody>
      {#each purchaseOrder as { itemName, unitType, unitPrice, quantity }} {#if
      unitPrice == 0 && quantity == 0}
      <tr>
        <th colspan="5">{$t[itemName]}</th>
      </tr>
      {:else}
      <tr>
        <td>{$t[itemName]}</td>
        <td>{unitType}</td>
        <td>{currency(unitPrice)}</td>
        <td>{quantity}</td>
        <td>{currency(unitPrice * quantity)}</td>
      </tr>
      {/if} {/each}
    </tbody>
    <tfoot>
      <tr>
        <th colspan="4">{$t.totalAmount}:</th>
        <th>{currency(totalCost)}</th>
      </tr>
      <tr>
        <th colspan="4">{$t.amountPaid}:</th>
        <th style="color: var(--green)">{currency(totalPaid)}</th>
      </tr>
      <tr>
        <th colspan="4">{$t.amountRemaining}:</th>
        <th style="color: var(--red)">{currency(totalCost - totalPaid)}</th>
      </tr>
    </tfoot>
  </table>
</div>
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import icons from "$lib/icons";

  export let labelKey: keyof typeof icons;
</script>

<span>
  <i class="bi {icons[labelKey]}" />
  {$t[labelKey]}
</span>
```

```ts
import type { BankAccount } from "$lib/types";

export const prices = {
  EASY_RIDER: 4500000,
  SELF_DRIVE: 3700000,
};

export const bankAccounts: { [key: string]: BankAccount } = {
  namco: {
    nickname: "Công ty Namco",
    accountName: "string",
    accountNumber: "string",
    bankName: "Vietcombank",
  },
  timo: {
    nickname: "Timo Account",
    accountName: "Tràn Quốc Hung",
    accountNumber: "9021059060951",
    bankName: "Viet Capital Bank",
  },
};
```

```ts
import { addDays, parseISO } from "date-fns";
import { formatDate } from "$lib/utils";
import type { Vendor, Tour } from "$lib/types";

function addDaysAndFormatDate(tour: Tour, days: number) {
  return formatDate(addDays(parseISO(tour.tourDate), days));
}

const vendors: Vendor[] = [
  {
    vendorId: "1",
    name: "🛏️ Hà Giang Hostel",
    bankAccount: {
      nickname: "Hà Giang Hostel",
      accountName: "Nguyen Van Viet",
      accountNumber: "34510000343489",
      bankName: "BIDV",
    },
    serviceSummary: {
      headers: ["date", "guests", "motorbike"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 0),
        tour.totalGuests,
        tour.selfDrive,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "singleBed",
        unitType: "",
        unitPrice: 200000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "breakfast",
        unitType: "",
        unitPrice: 0,
        quantity: tour.totalGuests,
      },
      {
        itemName: "hondaWave",
        unitType: "200.000 x 3 Ngày",
        unitPrice: 600000,
        quantity: tour.selfDrive,
      },
      {
        itemName: "extraProtection",
        unitType: "100.000 x 3 Ngày",
        unitPrice: 300000,
        quantity: tour.selfDrive,
      },
      {
        itemName: "fullFuelTank",
        unitType: "80.000 x 3 Ngày",
        unitPrice: 240000,
        quantity: tour.selfDrive,
      },
      {
        itemName: "insurance",
        unitType: "80.000 x 3 Ngày",
        unitPrice: 240000,
        quantity: tour.selfDrive,
      },
    ],
  },
  {
    vendorId: "2",
    name: "🕵️‍♂️ Ket (Lái chính trong đoàn)",
    bankAccount: {
      nickname: "Ket (Lái chính trong đoàn)",
      accountName: "DINH VAN KET",
      accountNumber: "199538366666",
      bankName: "Military Bank",
    },
    serviceSummary: {
      headers: ["date", "easyRider", "selfDrive"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 1),
        tour.easyRider,
        tour.selfDrive,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "coDriver",
        unitType: "",
        unitPrice: 0,
        quantity: 0,
      },
      {
        itemName: "driverHireThreeDays",
        unitType: "",
        unitPrice: 1500000,
        quantity: tour.easyRider,
      },
      {
        itemName: "meals",
        unitType: "50.000 x 7",
        unitPrice: 350000,
        quantity: tour.easyRider,
      },
      {
        itemName: "fuel",
        unitType: "",
        unitPrice: 200000,
        quantity: tour.easyRider,
      },
      {
        itemName: "activities",
        unitType: "",
        unitPrice: 0,
        quantity: 0,
      },
      {
        itemName: "lungKhuyCaveTicket",
        unitType: "",
        unitPrice: 30000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "kingMeoMansionFee",
        unitType: "",
        unitPrice: 30000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "lungCuFlagpoleParkingFee",
        unitType: "",
        unitPrice: 30000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "valleyBoatFee",
        unitType: "",
        unitPrice: 120000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "additionalFees",
        unitType: "",
        unitPrice: 0,
        quantity: 0,
      },
      {
        itemName: "additionalDriverHire",
        unitType: "",
        unitPrice: 500000,
        quantity: tour.selfDrive,
      },
      {
        itemName: "fuel",
        unitType: "200.000 - 80.000",
        unitPrice: 120000,
        quantity: tour.selfDrive,
      },
    ],
  },
  {
    vendorId: "3",
    name: "🛏️ Đồng Văn B&B",
    bankAccount: {
      nickname: "Đồng Văn B&B",
      accountName: "vũ lệ thu",
      accountNumber: "105896116666",
      bankName: "Vietnbank",
    },
    serviceSummary: {
      headers: ["date", "guests", "vegetarian", "easyDriver"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 1),
        tour.totalGuests,
        tour.vegetarian,
        tour.easyRider,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "singleBedForGuest",
        unitType: "",
        unitPrice: 100000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "breakfastForGuest",
        unitType: "",
        unitPrice: 50000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "dinnerForGuest",
        unitType: "",
        unitPrice: 100000,
        quantity: tour.totalGuests,
      },
      {
        itemName: "singleBedForDriver",
        unitType: "",
        unitPrice: 100000,
        quantity: tour.easyRider,
      },
      {
        itemName: "breakfastForDriver",
        unitType: "",
        unitPrice: 0,
        quantity: tour.easyRider,
      },
      {
        itemName: "dinnerForDriver",
        unitType: "",
        unitPrice: 50000,
        quantity: tour.easyRider,
      },
    ],
  },
  {
    vendorId: "4",
    name: "🛏️ Little Yen's Homestay",
    bankAccount: {
      nickname: "Little Yen's Homestay",
      accountName: "SUNG MY YEN",
      accountNumber: "8203215024545",
      bankName: "Agribank",
    },
    serviceSummary: {
      headers: ["date", "guests", "vegetarian", "easyRider"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 2),
        tour.totalGuests,
        tour.vegetarian,
        tour.easyRider,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "discountedSingleBedForGuest",
        unitType: "200.000 - 15% = 170.000",
        unitPrice: 170000,
        quantity: tour.totalGuests,
        totalAmount: 1190000,
      },
      {
        itemName: "breakfastForGuest",
        unitType: "",
        unitPrice: 0,
        quantity: tour.totalGuests,
        totalAmount: 0,
      },
      {
        itemName: "dinnerForGuest",
        unitType: "",
        unitPrice: 100000,
        quantity: tour.totalGuests,
        totalAmount: 700000,
      },
      {
        itemName: "freeSingleBedForDriver",
        unitType: "miễn phí cho tài xế",
        unitPrice: 0,
        quantity: Math.min(tour.easyRider, 2),
        totalAmount: 0,
      },
      {
        itemName: "chargedSingleBedForDriver",
        unitType: "",
        unitPrice: 100000,
        quantity: tour.easyRider - Math.min(tour.easyRider, 2),
        totalAmount: 400000,
      },
      {
        itemName: "breakfastForDriver",
        unitType: "",
        unitPrice: 0,
        quantity: tour.easyRider,
        totalAmount: 0,
      },
      {
        itemName: "dinnerForDriver",
        unitType: "",
        unitPrice: 80000,
        quantity: tour.easyRider,
        totalAmount: 480000,
      },
    ],
  },
  {
    vendorId: "5",
    name: "🍜 Yên Minh Quán",
    bankAccount: {
      nickname: "Yên Minh Quán",
      accountName: "Trần Thị Vân Anh",
      accountNumber: "106826031986",
      bankName: "Viettinbank",
    },
    serviceSummary: {
      headers: ["date", "eatsMeat", "vegetarian"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 1),
        tour.totalGuests - tour.vegetarian,
        tour.vegetarian,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "lunchForGuest",
        unitType: "",
        unitPrice: 70000,
        quantity: tour.totalGuests,
        totalAmount: 490000,
      },
    ],
  },
  {
    vendorId: "6",
    name: "🍜 Đồng Văn Quán",
    bankAccount: {
      nickname: "Đồng Văn Quán",
      accountName: "PHAM VAN TIEU",
      accountNumber: "8202205002572",
      bankName: "Agribank",
    },
    serviceSummary: {
      headers: ["date", "eatsMeat", "vegetarian"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 2),
        tour.totalGuests - tour.vegetarian,
        tour.vegetarian,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "lunchForGuest",
        unitType: "",
        unitPrice: 80000,
        quantity: tour.totalGuests,
        totalAmount: 560000,
      },
    ],
  },
  {
    vendorId: "7",
    name: "🍜 T.T Pác Miầu Quán",
    bankAccount: {
      nickname: "T.T Pác Miầu Quán",
      accountName: "Ma Thị Quy",
      accountNumber: "8301205062632",
      bankName: "Agribank",
    },
    serviceSummary: {
      headers: ["date", "eatsMeat", "vegetarian"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 3),
        tour.totalGuests - tour.vegetarian,
        tour.vegetarian,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "lunchForGuest",
        unitType: "",
        unitPrice: 70000,
        quantity: tour.totalGuests,
        totalAmount: 490000,
      },
    ],
  },
  {
    vendorId: "8",
    name: "🍜 Mộc Miên Quán",
    bankAccount: {
      nickname: "Mộc Miên Quán",
      accountName: "NGUYEN DUC MANH",
      accountNumber: "31000276480",
      bankName: "VietcomBank",
    },
    serviceSummary: {
      headers: ["date", "eatsMeat", "vegetarian"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        addDaysAndFormatDate(tour, 3),
        tour.totalGuests - tour.vegetarian,
        tour.vegetarian,
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "dinnerForGuest",
        unitType: "",
        unitPrice: 83000,
        quantity: tour.totalGuests,
        totalAmount: 581000,
      },
    ],
  },
  {
    vendorId: "9",
    name: "🚌 Nhà Xe Quang Nghị",
    bankAccount: {
      nickname: "Nhà Xe Quang Nghị",
      accountName: "Nguyen van nghi",
      accountNumber: "104001603289",
      bankName: "VietinBank",
    },
    serviceSummary: {
      headers: ["guests", "depart", "return"],
      rowTransformer: (tour: Tour) => [
        tour.id,
        tour.totalGuests,
        addDaysAndFormatDate(tour, 0),
        addDaysAndFormatDate(tour, 3),
      ],
    },
    purchaseOrder: (tour: Tour) => [
      {
        itemName: "bedBusHanoiToHaGiang",
        unitType: "Chỗ",
        unitPrice: 225000,
        quantity: tour.totalGuests,
        totalAmount: 1575000,
      },
      {
        itemName: "bedBusHaGiangToHanoi",
        unitType: "Chỗ",
        unitPrice: 225000,
        quantity: tour.totalGuests,
        totalAmount: 1575000,
      },
    ],
  },
];

export default vendors;
```

```ts
export default {
  home: "🏠 Home",
  itemName: "📦 Item Name",
  unitType: "📏 Unit Type",
  unitPrice: "💰 Unit Price",
  quantity: "🔢 Quantity",
  totalAmount: "💲 Total Amount",
  date: "📅 Date",
  eatsMeat: "🥩 Eats Meat",
  vegetarian: "🌱 Vegetarian",
  guestDiet: "🍽️ Guest Diet",
  dateIssued: "📅 Date Issued",
  drivingType: "🏍️ Driving Type",
  selfDrive: "🏍️ Self Drive",
  easyRider: "🏍️ Easy Driver",
  easyDriver: "🏍️ Driver",
  motorbike: "🏍️ Motorbike",
  totalGuests: "🧑‍🤝‍🧑 Total Guests",
  guests: "🧑‍🤝‍🧑 Guests",
  nickname: "🆔 Nickname",
  accountName: "🏦 Account Name",
  accountNumber: "🔢 Account Number",
  bankName: "🏛️ Bank Name",
  organisingUnit: "🛒 Merchant",
  vendor: "🛒 Vendor",
  purchaseOrder: "📝 Purchase Order",
  serviceSummary: "📊 Service Summary",
  serviceDescription: "📋 Service Description",
  tour: "🌄 Tour",
  booking: "🎟️ Bookings",
  createBooking: "🎟️ Create Booking",
  back: "Back",
  bookingId: "🆔 Booking ID",
  tourDate: "📅 Tour Date",
  tourId: "🌄 Tour ID",
  guestName: "🧑‍🤝‍🧑 Guest Name",
  drivingChoice: "🏍️ Driving Choice",
  hasGivenPassport: "🛂 Passport Provided",
  hasPaid: "💳 Paid",
  whatsApp: "📱 WhatsApp",
  email: "📧 Email",
  guestList: "🧑‍🤝‍🧑 Guest List",
  depart: "➡️ Depart",
  return: "↩️ Return",
  transaction: "💸 Transaction",
  amountRemaining: "❌ Amount Remaining",
  amountPaid: "✅ Amount Paid",
  finance: "📈 Finance",
  pay: "💳 Pay",
  // Vendor 1
  singleBed: "🛏️ Single Bed for 1 person",
  breakfast: "🍳 Breakfast",
  hondaWave: "🏍️ Honda Wave 110cc",
  extraProtection: "🛡️ Extra Protection",
  fullFuelTank: "⛽ Full fuel tank",
  insurance: "📜 Insurance",
  // Vendor 2
  coDriver: "🧑‍✈️ Co-Driver",
  driverHireThreeDays: "🚗 3 Days Driver Hire + Driving",
  meals: "🍲 Meals",
  fuel: "⛽ Vehicle Fuel",
  activities: "🎟️ Activities",
  lungKhuyCaveTicket: "🕳️ Lùng Khuý Cave Entrance Fee",
  kingMeoMansionFee: "🏰 King Meo Mansion Entrance Fee",
  lungCuFlagpoleParkingFee: "🚩 Lũng Cú Flagpole Parking Fee",
  valleyBoatFee: "🚤 Valley Boat Fee",
  additionalFees: "💰 Additional Fees",
  additionalDriverHire: "🧑‍✈️ Additional Driver Hire",
  // Vendor 3
  singleBedForGuest: "🛏️ Single Bed for 1 guest",
  breakfastForGuest: "🍳 Breakfast for guest",
  dinnerForGuest: "🍽️ Dinner for guest",
  singleBedForDriver: "🛏️ Single Bed for 1 driver",
  breakfastForDriver: "🍳 Breakfast for driver",
  dinnerForDriver: "🍽️ Dinner for driver",
  // Vendor 4
  discountedSingleBedForGuest: "🛏️ Single Bed for 1 guest (Discounted)",
  freeSingleBedForDriver: "🛏️ Single Bed for 1 driver (Free for driver)",
  chargedSingleBedForDriver: "🛏️ Single Bed for 1 driver",
  // Vendor 5
  lunchForGuest: "🍽️ Lunch for guest",
  // Vendor 9
  bedBusHanoiToHaGiang: "🚌 Sleeper bus from Hanoi to Ha Giang",
  bedBusHaGiangToHanoi: "🚌 Sleeper bus from Ha Giang to Hanoi",
};
```

```ts
export default {
  home: "🏠 Trang chủ",
  itemName: "📦 Danh Mục",
  unitType: "📏 Quy cách",
  unitPrice: "💰 Đơn Giá",
  quantity: "🔢 Số Lượng",
  totalAmount: "💲 Thành tiền",
  date: "📅 Ngày",
  eatsMeat: "🥩 Người ăn thịt",
  vegetarian: "🌱 Người ăn chay",
  guestDiet: "🍽️ Khách ăn kiêng",
  dateIssued: "📅 Ngày gửi",
  drivingType: "🏍️ Loại lái xe",
  selfDrive: "🏍️ Tự Lái",
  easyRider: "🏍️ Hỗ trợ lái",
  easyDriver: "🏍️ Lái Phụ",
  motorbike: "🏍️ Xe máy",
  totalGuests: "🧑‍🤝‍🧑 Tổng số khách",
  guests: "🧑‍🤝‍🧑 Khách",
  nickname: "🆔 Tên nick",
  accountName: "🏦 Tên tài khoản",
  accountNumber: "🔢 Số tài khoản",
  bankName: "🏛️ Tên ngân hàng",
  organisingUnit: "📋 Đơn vị tổ chức",
  vendor: "🛒 Người bán",
  purchaseOrder: "📝 Đơn đặt hàng",
  serviceSummary: "📊 Tóm tắt dịch vụ",
  serviceDescription: "📋 Chi tiết dịch vụ",
  tour: "🌄 Chuyến Du Lịch",
  booking: "🎟️ Đặt Chỗ",
  createBooking: "🎟️ Tạo đặt chỗ",
  back: "Quay lại",
  bookingId: "🆔 Mã Đặt Chỗ",
  tourDate: "📅 Ngày Du Lịch",
  tourId: "🌄 Mã Chuyến Du Lịch",
  guestName: "🧑‍🤝‍🧑 Tên Khách",
  drivingChoice: "🏍️ Lựa Chọn Lái Xe",
  hasGivenPassport: "🛂 Đã Cung Cấp Hộ Chiếu",
  hasPaid: "💳 Đã Thanh Toán",
  whatsApp: "📱 WhatsApp",
  email: "📧 Email",
  guestList: "🧑‍🤝‍🧑 Danh sách",
  depart: "➡️ Khởi hành",
  return: "↩️ Trở về",
  transaction: "💸 Giao dịch",
  amountRemaining: "❌ Số tiền còn lại",
  amountPaid: "✅ Số tiền đã trả",
  finance: "📈 Tài chính",
  pay: "💳 Thanh toán",
  // Vendor 1
  singleBed: "🛏️ Giường Đơn cho 1 người",
  breakfast: "🍳 Bữa sáng",
  hondaWave: "🏍️ Honda Wave 110cc",
  extraProtection: "🛡️ Bảo vệ extra",
  fullFuelTank: "⛽ Bình xăng đầy",
  insurance: "📜 Bảo hiểm",
  // Vendor 2
  coDriver: "🧑‍✈️ Lái Phụ",
  driverHireThreeDays: "🚗 Chi phí thuê Tài xế 3 ngày + Lái",
  meals: "🍲 Bữa ăn",
  fuel: "⛽ Xăng Xe",
  activities: "🎟️ Các hoạt động",
  lungKhuyCaveTicket: "🕳️ Vé vào hang Lùng Khuý",
  kingMeoMansionFee: "🏰 Phí tham quan dinh thự Vua Mèo",
  lungCuFlagpoleParkingFee: "🚩 Phí gửi xe tại cột cờ Lũng Cú",
  valleyBoatFee: "🚤 Phí đi thuyền tại Thung Lũng",
  additionalFees: "💰 Phụ Phí",
  additionalDriverHire: "🧑‍✈️ Chi phí thuê thêm Tài xế",
  // Vendor 3
  singleBedForGuest: "🛏️ Giường Đơn cho 1 khách",
  breakfastForGuest: "🍳 Bữa sáng cho khách",
  dinnerForGuest: "🍽️ Bữa tối cho khách",
  singleBedForDriver: "🛏️ Giường Đơn cho 1 người lái xe",
  breakfastForDriver: "🍳 Bữa sáng cho người lái xe",
  dinnerForDriver: "🍽️ Bữa tối cho người lái xe",
  // Vendor 4
  discountedSingleBedForGuest: "🛏️ Giường Đơn cho 1 khách (Giảm giá)",

  freeSingleBedForDriver:
    "🛏️ Giường Đơn cho 1 người lái xe (Miễn phí cho tài xế)",
  chargedSingleBedForDriver: "🛏️ Giường Đơn cho 1 người lái xe",
  // Vendor 5
  lunchForGuest: "🍽️ Bữa Trưa cho khách",
  // Vendor 0
  bedBusHanoiToHaGiang: "🚌 Xe giường nằm từ Hà Nội đi Hà Giang",
  bedBusHaGiangToHanoi: "🚌 Xe giường nằm từ Hà Giang đi Hà Nội",
};
```

```ts
const icons = {
  home: "bi-house",
  tour: "bi-compass",
  booking: "bi-calendar",
  createBooking: "bi-plus-circle",
  transaction: "bi-cash",
  finance: "bi-currency-dollar",
  serviceSummary: "bi-file-earmark-bar-graph",
  vendor: "bi-building",
};

export default icons;
```

```ts
import { derived, writable } from "svelte/store";

import en from "./locales/en.js";
import vi from "./locales/vi.js";

export const locales = [
  {
    name: "English",
    isoLanguageCode: "en",
    isoCountryCode: "GB",
    emoji: "🇬🇧",
    data: en,
  },
  {
    name: "Tiếng Việt",
    isoLanguageCode: "vi",
    isoCountryCode: "VN",
    emoji: "🇻🇳",
    data: vi,
  },
];

export let locale = writable("vi");

let t = derived(
  locale,
  ($locale) =>
    locales.find((l) => l.isoLanguageCode == $locale)?.data || locales[0].data
);

export default t;

export let displayCurrency = writable("GBP");
```

```ts
import type { Tour, Transaction } from "$lib/types";
import initDb from "$lib/client/db";

async function getTable(tableName: string) {
  const db = await initDb();
  const queryResult = await db.query(`SELECT * FROM ${tableName}`);
  return queryResult[0].result;
}

export async function getTours(): Promise<Tour[]> {
  const tours = await getTable("tour");
  // @ts-ignore
  return tours;
}

export async function getTransactions(): Promise<Transaction[]> {
  const transactions = await getTable("transaction");
  // @ts-ignore
  return transactions;
}

export async function getTourDateOptions(): Promise<[]> {
  const tourDateOptions = await getTable("tourDateOption");
  // @ts-ignore
  return tourDateOptions;
}
```

```ts
export type Tour = {
  id: string;
  tourDate: date;
  totalGuests: number;
  easyRider: number;
  selfDrive: number;
  vegetarian: number;
  whatsappGroupId: string | null;
  guests: Guest[];
};

export type TourDateOption = {
  tourId: string | null; // "tours:nf57k31qzknarj412z44"
  tourDate: string; // "2023-06-09T00:00:00Z"
};

export type Guest = {
  bookingId;
  guestName: string;
  drivingChoice: "selfDrive" | "easyRider";
  isVegetarian: boolean;
  hasGivenPassport: boolean;
  hasPaid: boolean;
  whatsApp: string;
  email: string;
};

type BankAccount = {
  nickname: string;
  accountName: string;
  accountNumber: string;
  bankName: string;
};

type ServiceSummary = {
  headers: string[];
  rowTransformer: (tour: Tour) => Array;
};

type PurchaseOrderRow = {
  itemName: string;
  unitType: string;
  unitPrice: number;
  quantity: number;
};

export type Vendor = {
  vendorId: string;
  name: string;
  bankAccount: BankAccount;
  serviceSummary: ServiceSummary;
  purchaseOrder: (tour: Tour) => PurchaseOrderRow[];
};

export type Transaction = {
  amount: number;
  receivingVendorId: string;
  forTourId: string;
};

export type UnconfirmedCheckout = {
  tourDateOption: TourDateOption;
  newGuests: Guest[];
  vnp_Amount: number;
  confirmed: boolean;
};
```

```ts
import { format, getDate, parseISO } from "date-fns";

export function currency(amount: number) {
  return new Intl.NumberFormat("vi-VN", {
    currency: "VND",
  }).format(amount);
}

function getOrdinalNum(number: number) {
  let selector;

  if (number <= 0) {
    selector = 4;
  } else if ((number > 3 && number < 21) || number % 10 > 3) {
    selector = 0;
  } else {
    selector = number % 10;
  }

  return number + ["th", "st", "nd", "rd", ""][selector];
}

export function formatDate(date: string | Date, formatOption?: "English") {
  const dateObject = typeof date === "string" ? parseISO(date) : date;

  if (formatOption == "English") {
    return format(
      dateObject,
      `EEEE 'the' '${getOrdinalNum(getDate(dateObject))}' 'of' MMMM`
    );
  }
  return format(dateObject, "dd/MM/yy");
}
```

```ts
import type { PageServerLoad } from "./$types";
import { getTourDateOptions } from "$lib/queryFunction";

export const load: PageServerLoad = async () => {
  const tourDateOptions = await getTourDateOptions();
  return { tourDateOptions };
};
```

```html
<script lang="ts">
  import TourDate from "./TourDate.svelte";
  import Guest from "./Guest.svelte";
  import Invoice from "./Invoice.svelte";
  import { goto } from "$app/navigation";
  import type { TourDateOption } from "$lib/types";
  import type { PageData } from "./$types";
  import { locale } from "$lib/localise";

  $locale = "en";

  export let data: PageData;

  let passcode: string;

  let selectedTourDateOption: TourDateOption;
  let invoiceTotal = 0;
  let process: boolean = false;

  const newGuest = () => ({
    bookingId: "",
    guestName: null,
    drivingChoice: null,
    isVegetarian: false,
    hasGivenPassport: false,
    hasPaid: true,
    whatsApp: null,
    email: null,
  });

  $: newGuests = [newGuest()];

  let form: HTMLFormElement;

  async function book() {
    if (!form.checkValidity()) {
      form.preventDefault();
      form.stopPropagation();
    }

    process = true;
    let res = await fetch("/book", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        tourDateOption: selectedTourDateOption,
        newGuests,
      }),
    });

    let paymentUrl = await res.json().then((d) => d.paymentUrl);
    goto(paymentUrl);
  }
</script>

<TourDate tourDateOptions={data.tourDateOptions} bind:selectedTourDateOption />

{#if selectedTourDateOption}
  <form
    bind:this={form}
    on:submit|preventDefault={book}
    class="col"
    data-testid="booking-form"
    autocomplete="off"
  >
    <hr class="my-4" />
    {#each newGuests as guest, index}
      <div data-testid="guest-element-{index}">
        <Guest bind:guest />
      </div>
      {#if index > 0}
        <button
          on:click={() => {
            newGuests.splice(index, 1), (newGuests = newGuests);
          }}
          type="button"
          data-testid="remove-guest-button-{index}"
        >
          Remove
        </button>
      {/if}

      <hr />
    {/each}
    <div class="row" style="justify-content: center;">
      <button
        type="button"
        class="bold"
        on:click={() => (newGuests = [...newGuests, newGuest()])}
        data-testid="add-guest-button"
      >
        <i class="fa-solid fa-user-plus" />
        Add another guest
      </button>
    </div>

    <label>
      Passcode
      <input
        data-testid="passcode"
        style="max-width: 10rem"
        bind:value={passcode}
      />
    </label>

    {#if passcode == "123"}
      <Invoice bind:newGuests bind:invoiceTotal />

      <div class="row" style="justify-content: center;">
        <button
          disabled={invoiceTotal == 0 || process}
          class="bold"
          style="width: 80%"
          type="submit"
          data-testid="submit-button"
        >
          {#if process}
            <i class="fa fa-spinner fa-spin" />
            <span role="status">Loading...</span>
          {:else}
            <i class="fa-solid fa-credit-card" />
            Checkout
          {/if}
        </button>
      </div>
    {/if}
  </form>
{/if}
```

```ts
import type { RequestHandler } from "./$types";
import type { RequestEvent } from "@sveltejs/kit";
import { prices } from "$lib/data/constants";
import type { Guest, UnconfirmedCheckout } from "$lib/types";
import initDb from "$lib/client/db";
import initVnpay from "$lib/client/vnpay";

const db = await initDb();
const vnpay = await initVnpay();

export const POST: RequestHandler = async (event: RequestEvent) => {
  const { tourDateOption, newGuests } = await event.request.json();

  const vnp_Amount = newGuests.reduce(
    (total: number, guest: Guest) =>
      total +
      (guest.drivingChoice === "easyRider"
        ? prices.EASY_RIDER
        : prices.SELF_DRIVE),
    0
  );

  const unconfirmedCheckout: UnconfirmedCheckout = {
    tourDateOption,
    newGuests,
    vnp_Amount,
    confirmed: false,
  };

  const { id } = await db.create("unconfirmedCheckout", unconfirmedCheckout);

  // ! Uncomment this when payment gateway is ready
  // const paymentUrl = await vnpay.buildPaymentUrl({
  //   vnp_Amount,
  //   vnp_IpAddr: event.getClientAddress(),
  //   vnp_TxnRef: id,
  //   vnp_OrderInfo: "Payment",
  // });

  // ! Remove this when payment gateway is ready
  const paymentUrl = `/book/confirm?vnp_Amount=${vnp_Amount}&vnp_TxnRef=${id}`;

  return new Response(JSON.stringify({ paymentUrl }), {
    status: 200,
  });
};
```

```html
<script lang="ts">
  import t from "$lib/localise";

  export let guest = {
    bookingId: "",
    guestName: null,
    drivingChoice: null,
    isVegetarian: false,
    hasGivenPassport: false,
    hasPaid: true,
    whatsApp: null,
    email: null,
  };
</script>

<div class="row" style="row-gap: 1rem;">
  <label>
    Full Name
    <input
      data-testid="name"
      type="text"
      style="max-width: 20rem"
      placeholder="First Last"
      bind:value="{guest.guestName}"
      required
    />
    <small>As shown on passport</small>
  </label>

  <label>
    WhatsApp
    <input
      data-testid="whatsapp"
      type="tel"
      style="width: 10rem"
      placeholder="+00 0000000000"
      bind:value="{guest.whatsApp}"
      required
    />
    <small>Not +84 number</small>
  </label>

  <label>
    Email
    <input
      data-testid="email"
      type="email"
      style="width: 15rem"
      placeholder="name@domain.com"
      bind:value="{guest.email}"
    />
    <small>.</small>
  </label>

  <label>
    Driving Choice
    <select
      data-testid="driving-choice"
      bind:value="{guest.drivingChoice}"
      required
    >
      <option value="selfDrive">{$t.selfDrive}</option>
      <option value="easyRider">{$t.easyRider}</option>
    </select>
    <small>.</small>
  </label>
  <div class="checkboxes">
    <label>
      <input
        data-testid="vegetarian"
        type="checkbox"
        bind:checked="{guest.isVegetarian}"
      />
      Vegetarian
    </label>
  </div>
</div>
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import { currency } from "$lib/utils";
  import { prices } from "$lib/data/constants";
  import type { Guest } from "$lib/types";

  export let newGuests: Guest[] = [];

  $: invoiceItems = [
    {
      itemName: $t.easyRider,
      unitPrice: prices.EASY_RIDER,
      quantity: newGuests.filter((g) => g.drivingChoice === "easyRider").length,
    },
    {
      itemName: $t.selfDrive,
      unitPrice: prices.SELF_DRIVE,
      quantity: newGuests.filter((g) => g.drivingChoice === "selfDrive").length,
    },
  ];

  export let invoiceTotal = 0;

  $: invoiceTotal = invoiceItems.reduce(
    (accumulator, item) => accumulator + item.unitPrice * item.quantity,
    0
  );
</script>

<div style="table-responsive">
  <table>
    <thead>
      <tr>
        <th>{$t.itemName}</th>
        <th>{$t.unitPrice}</th>
        <th>{$t.quantity}</th>
        <th>{$t.totalAmount}</th>
      </tr>
    </thead>
    <tbody>
      {#each invoiceItems as { itemName, unitPrice, quantity }}
      <tr>
        <th>{itemName}</th>
        <td>{currency(unitPrice)}</td>
        <td>{quantity}</td>
        <td>{currency(unitPrice * quantity)}</td>
      </tr>
      {/each}
    </tbody>
    <tfoot>
      <tr>
        <th />
        <th />
        <th>{$t.totalAmount}:</th>
        <th data-testid="invoice-total-amount">{currency(invoiceTotal)}</th>
      </tr>
    </tfoot>
  </table>
</div>
```

```html
<script lang="ts">
  import { isPast, isToday, parseISO, compareAsc } from "date-fns";
  import { formatDate } from "$lib/utils";
  import type { TourDateOption } from "$lib/types";

  export let tourDateOptions: TourDateOption[];
  export let selectedTourDateOption: TourDateOption;

  const sortedUpcomingTourDateOptions = tourDateOptions
    .filter(
      (tour) =>
        !isPast(parseISO(tour.tourDate)) || isToday(parseISO(tour.tourDate))
    )
    .sort((a, b) => compareAsc(parseISO(a.tourDate), parseISO(b.tourDate)));
</script>

{#if sortedUpcomingTourDateOptions.length === 0}
<div class="message">📅 There are no available tour dates</div>
{:else}
<div>
  <label>
    Start Date
    <select
      bind:value="{selectedTourDateOption}"
      required
      data-testid="tour-date-select"
    >
      {#each sortedUpcomingTourDateOptions as tourDateOption}
      <option value="{tourDateOption}">
        {formatDate(tourDateOption.tourDate)} -
        {formatDate(tourDateOption.tourDate, "English")}
      </option>
      {/each}
    </select>
  </label>
</div>
{/if}
```

```ts
import type { PageServerLoad } from "./$types";
import { getTours } from "$lib/queryFunction";

export const load: PageServerLoad = async () => {
  const tours = await getTours();
  return { tours };
};
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import { formatDate } from "$lib/utils";
  import type { PageData } from "./$types";

  export let data: PageData;

  function boolToEmoji(bool: boolean) {
    return bool ? "✅" : "❌";
  }

  function drivingTypeFormat(drivingType: string) {
    return drivingType == "selfDrive" ? "🏍️ Self Drive" : "🛟 Easy Rider";
  }

  let selectedTourDate = new Date();

  $: guests =
    data.tours.find((tour) => tour.tourDate == selectedTourDate)?.guests || [];
</script>

<h2>{$t.booking}</h2>

<div class="row mb-3">
  <div class="col">
    <label>{$t.tourDate}</label>
    <select class="form-select" bind:value="{selectedTourDate}">
      {#each data.tours as { tourDate }}
      <option value="{tourDate}">{formatDate(tourDate)}</option>
      {/each}
    </select>
  </div>
</div>

{#if guests.length > 0}
<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th>{$t.guestName}</th>
        <th>{$t.drivingChoice}</th>
        <th>{$t.hasGivenPassport}</th>
        <th>{$t.hasPaid}</th>
        <th>{$t.whatsApp}</th>
        <th>{$t.email}</th>
        <th>{$t.vegetarian}</th>
      </tr>
    </thead>

    <tbody>
      {#each guests as { bookingId, guestName, drivingChoice, hasGivenPassport,
      hasPaid, whatsApp, email, isVegetarian }}
      <tr>
        <td class="text-nowrap">{guestName.toUpperCase()}</td>
        <td class="text-nowrap">{drivingTypeFormat(drivingChoice)}</td>
        <td>{boolToEmoji(hasGivenPassport)}</td>
        <td>{boolToEmoji(hasPaid)}</td>
        <td class="text-nowrap">{whatsApp}</td>
        <td class="text-nowrap"><a href="{`mailto:${email}`}">{email}</a></td>
        <td>{boolToEmoji(isVegetarian)}</td>
      </tr>
      {/each}
    </tbody>
  </table>
</div>
{/if}

<style>
  td {
    min-width: 6rem;
  }
</style>
```

```ts
import type { PageServerLoad } from "../$types";
import vendors from "$lib/data/vendors";
import { getTours, getTransactions } from "$lib/queryFunction";

export const load: PageServerLoad = async ({ params }) => {
  const transactions = await getTransactions();
  const tours = await getTours();

  const tour = tours.find((tour) => tour.id == params.tourId);

  const vendor = vendors.find((vendor) => vendor.vendorId == params.vendorId);

  return {
    vendorId: vendor.vendorId,
    guests: tour?.guests || [],
    purchaseOrder: vendor.purchaseOrder(tour),
    transactions: transactions.filter(
      (t) =>
        t.receivingVendorId == params.vendorId && t.forTourId == params.tourId
    ),
    tour,
  };
};
```

```html
<script lang="ts">
  import { formatDate } from "$lib/utils";
  import { goto } from "$app/navigation";
  import t from "$lib/localise";
  import type { PageData } from "./$types";

  import GuestList from "$lib/components/GuestList.svelte";
  import PurchaseOrder from "$lib/components/PurchaseOrder.svelte";

  export let data: PageData;
</script>

<button
  on:click={() => goto(`/serviceSummary/${data.vendorId}`)}
  type="button"
  style="width: 100%"
>
  <i class="fa-solid fa-chevron-left" />
  {$t.back}
</button>

<div class="col spread">
  <h3>{$t.tour}</h3>
  <h4>{formatDate(data.tour?.tourDate)}</h4>

  <h3 id="purchaseOrder">{$t.purchaseOrder}</h3>
  <PurchaseOrder
    purchaseOrder={data.purchaseOrder}
    transactions={data.transactions}
  />

  <hr style="margin: 2rem 0;" />

  <h3 id="guestList">{$t.guestList}</h3>
  <GuestList guests={data.guests} />
</div>

```

```html
<script lang="ts">
  import vendors from "$lib/data/vendors";
  import { currency } from "$lib/utils";
  import type { Tour } from "$lib/types";
  import t from "$lib/localise";

  function createTour(easyRider: number, selfDrive: number) {
    return {
      totalGuests: easyRider + selfDrive,
      easyRider,
      selfDrive,
    };
  }

  function totalCostForTour(tour: Tour) {
    return vendors
      .map((vendor) =>
        vendor
          .purchaseOrder(tour)
          .reduce(
            (accumulator, item) => accumulator + item.unitPrice * item.quantity,
            0
          )
      )
      .reduce((accumulator, item) => accumulator + item, 0);
  }

  $: easyRiderCount = 2;
  $: selfDriveCount = 5;

  $: tour = createTour(easyRiderCount, selfDriveCount);

  $: totalCostForVariableTour = totalCostForTour(tour);

  const fixedGroupLeaderCost = 500000;

  let easyRiderCost = totalCostForTour(createTour(1, 0)) - fixedGroupLeaderCost;
  let selfDriveCost = totalCostForTour(createTour(0, 1)) - fixedGroupLeaderCost;

  const netSalePriceAfterFees = {
    easyRider: 4900000,
    selfDrive: 4100000,
  };
</script>

<h2>{$t.finance}</h2>

<h4>Forecast</h4>

<div class="row">
  <label>
    Self Drive
    <input
      type="number"
      bind:value="{selfDriveCount}"
      style="max-width: 80px;"
    />
  </label>
  <label>
    Easy Rider
    <input
      type="number"
      bind:value="{easyRiderCount}"
      style="max-width: 80px;"
    />
  </label>
</div>

<dl>
  <dt>Total cost for tour:</dt>
  <dd>{currency(totalCostForVariableTour)}</dd>
</dl>

<hr />

<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>{$t.easyRider}</th>
        <th>{$t.selfDrive}</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>{currency(easyRiderCost)}</td>
        <td>{currency(selfDriveCost)}</td>
      </tr>
    </tbody>
  </table>
</div>

<p>Base cost price excluding 500.000 group leader bonus.</p>
```

```html
<img src="/img/ha-giang-hostel.png" />

<h4>Trách nhiệm</h4>
<div class="table-responsive">
  <table>
    <tbody>
      <tr>
        <th>Ha Giang Hostel</th>
        <td>
          <ol>
            <li>Xăng cần đổ đầy bình trước khi di chuyển</li>
          </ol>
        </td>
      </tr>
      <tr>
        <th>Tất cả trình điều khiển</th>
        <td>
          <ol>
            <li>Đổ xăng cho xe máy của họ trong chuyến đi</li>
            <li>Nạp nhiên liệu cho xe máy trước khi chuyến đi bắt đầu</li>
          </ol>
        </td>
      </tr>
      <tr>
        <th>Tài xế chính</th>
        <td>
          <ol>
            <li>Trả tiền xăng xe máy trong suốt chuyến đi</li>
            <li>Không trả tiền xăng cho chiếc xe máy thuê vào buổi sáng</li>
          </ol>
        </td>
      </tr>
      <tr>
        <th>Khách hàng</th>
        <td>Không chịu trách nhiệm về vấn đề xe máy</td>
      </tr>
    </tbody>
  </table>
</div>

<div class="table">
  <table>
    <thead>
      <tr>
        <th>Thời gian</th>
        <th />
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>Giờ vào</th>
        <td>18:00</td>
      </tr>
      <tr>
        <th>Bữa tối</th>
        <td>19:00</td>
      </tr>
      <tr>
        <th>Bữa sáng</th>
        <td>07:15</td>
      </tr>
      <tr>
        <th>Giờ ra</th>
        <td>09:00</td>
      </tr>
    </tbody>
  </table>
</div>

<h4>Chi phí đã chi trả cho tài xế phụ</h4>
<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>Danh Mục</th>
        <th>Số lượng</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1 đêm ở Giường Đơn</td>
        <td>2</td>
      </tr>
      <tr>
        <td>Ăn sáng tại nhà dân</td>
        <td>2</td>
      </tr>
      <tr>
        <td>Ăn tối tại nhà dân</td>
        <td>2</td>
      </tr>
    </tbody>
  </table>
</div>

<script>
  import Homestay from "$lib/components/ServiceDescription/Homestay.svelte";
</script>

<Homestay />

<script>
  import Homestay from "$lib/components/ServiceDescription/Homestay.svelte";
</script>

<Homestay />

<script>
  import Restaurant from "$lib/components/ServiceDescription/Restaurant.svelte";
</script>

<Restaurant />

<script>
  import Restaurant from "$lib/components/ServiceDescription/Restaurant.svelte";
</script>

<Restaurant />

<script>
  import Restaurant from "$lib/components/ServiceDescription/Restaurant.svelte";
</script>

<Restaurant />

<script>
  import Restaurant from "$lib/components/ServiceDescription/Restaurant.svelte";
</script>

<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>Thời gian</th>
        <th />
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>Bữa tối</th>
        <td>19:00</td>
      </tr>
      <tr>
        <th>Bus departs</th>
        <td>20:20</td>
      </tr>
    </tbody>
  </table>
</div>

<Restaurant />
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import { page } from "$app/stores";
  import vendors from "$lib/data/vendors";

  let vendorId = $page.route.id.split("/")[2];

  let vendor = vendors.find((vendor) => vendor.vendorId == vendorId);
</script>

<a type="button" style="width: 100%" href="/serviceSummary/{vendorId}">
  <i class="fa-solid fa-chevron-left" />
  {$t.serviceSummary}
</a>

<h3>{$t.serviceDescription}</h3>
<h4>{vendor?.name}</h4>

<slot />
```

```ts
import { error } from "@sveltejs/kit";
import type { PageServerLoad } from "./$types";
import type { Vendor } from "$lib/types";
import vendors from "$lib/data/vendors";
import { getTours } from "$lib/queryFunction";

export const load: PageServerLoad = async ({ params }) => {
  const vendor: Vendor | undefined = vendors.find(
    (vendor) => vendor.vendorId == params.vendorId
  );

  if (!vendor) {
    throw error(404, "Vendor not found");
  }

  const tours = await getTours();

  const rows = tours.map((tour) => vendor.serviceSummary.rowTransformer(tour));

  return {
    vendorId: vendor.vendorId,
    vendorName: vendor.name,
    headers: vendor?.serviceSummary.headers,
    rows,
  };
};
```

```html
<script lang="ts">
  import type { PageData } from "./$types";
  import t from "$lib/localise";
  import { goto } from "$app/navigation";

  export let data: PageData;
</script>

<h3>{$t.serviceSummary}</h3>

<a type="button" href="/serviceDescription/{data.vendorId}">
  {$t.serviceDescription}
</a>

<div style="table-responsive">
  <table style="width: 100%;">
    <thead>
      <tr>
        {#each data.headers as header}
          <th data-testid="header-{header}">{$t[header]}</th>
        {/each}
        <th />
      </tr>
    </thead>
    <tbody>
      {#each data.rows as row}
        <tr
          style="cursor: pointer"
          on:click={() => goto(`/detail/${data.vendorId}-${row[0]}`)}
        >
          {#each row.slice(1) as column}
            <td data-testid="column-{column}">{column}</td>
          {/each}
          <td>
            <i class="fa-solid fa-chevron-right" />
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

```

```ts
import type { PageServerLoad } from "./$types";
import { getTours } from "$lib/queryFunction";

export const load: PageServerLoad = async () => {
  const tours = await getTours();
  return { tours };
};
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import { formatDate } from "$lib/utils";
  import type { PageData } from "./$types";

  export let data: PageData;
</script>

<h2>{$t.tour}</h2>

<div style="table-responsive">
  <table>
    <thead>
      <tr>
        <th>{$t.date}</th>
        <th>{$t.totalGuests}</th>
        <th>{$t.easyRider}</th>
        <th>{$t.selfDrive}</th>
        <th>{$t.vegetarian}</th>
      </tr>
    </thead>
    <tbody>
      {#each data.tours as { tourDate, totalGuests, easyRider, selfDrive,
      vegetarian }}
      <tr>
        <td>{formatDate(tourDate, "English")}</td>
        <td>{totalGuests}</td>
        <td>{easyRider}</td>
        <td>{selfDrive}</td>
        <td>{vegetarian}</td>
      </tr>
      {/each}
    </tbody>
  </table>
</div>

<style>
  td {
    min-width: 5rem;
  }
</style>
```

```ts
import type { PageServerLoad } from "./$types";
import vendors from "$lib/data/vendors";
import { getTours, getTransactions } from "$lib/queryFunction";

export const load: PageServerLoad = async () => {
  const tours = await getTours();
  const transactions = await getTransactions();

  const enrichedTransactions = transactions.map((trans) => ({
    ...trans,
    ...vendors.find((vendor) => vendor.vendorId == trans.receivingVendorId)
      ?.bankAccount,
    tourDate: tours.find((tour) => tour.id == trans.forTourId)?.tourDate,
  }));

  return { transactions: enrichedTransactions };
};
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import { formatDate } from "$lib/utils";
  import { currency } from "$lib/utils";
  import type { PageData } from "./$types";

  export let data: PageData;
</script>

<h2>{$t.transaction}</h2>

<div class="table-responsive">
  <table>
    <thead>
      <tr>
        <th>{$t.tourDate}</th>
        <th>{$t.nickname}</th>
        <th>{$t.totalAmount}</th>
      </tr>
    </thead>

    <tbody>
      {#each data.transactions as { nickname, tourDate, amount }}
      <tr>
        <td style="width: 3rem">{formatDate(tourDate)}</td>
        <td class="text-nowrap">{nickname}</td>
        <th>{currency(amount)}</th>
      </tr>
      {/each}
    </tbody>
  </table>
</div>
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import type { Vendor } from "$lib/types";

  export let data: Vendor;
</script>

<h2>{$t.vendor}</h2>
<h3>{data.name}</h3>

<table class="table">
  <thead>
    <tr>
      <th>🏦 Bank Account</th>
      <th />
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>{$t.nickname}</th>
      <td>{data.bankAccount.nickname}</td>
    </tr>
    <tr>
      <th>{$t.accountName}</th>
      <td>{data.bankAccount.accountName}</td>
    </tr>
    <tr>
      <th>{$t.accountNumber}</th>
      <td>{data.bankAccount.accountNumber}</td>
    </tr>
    <tr>
      <th>{$t.bankName}</th>
      <td>{data.bankAccount.bankName}</td>
    </tr>
  </tbody>
</table>
```

```ts
import vendors from "$lib/data/vendors";

export function load({ params }) {
  const vendor = vendors.find((vendor) => vendor.vendorId == params.vendorId);

  return vendor;
}
```

```html
<script lang="ts">
  import { page } from "$app/stores";
</script>

<h3>Error {$page.status}</h3>
<p>{$page.error?.message}</p>

<button on:click={() => window.history.back()}>Go Back</button>

```

```html
<script lang="ts">
  import "../style/index.css";
  import LanguageSelection from "$lib/components/LanguageSelection.svelte";
</script>

<main class="col" style="row-gap: 2rem;">
  <header class="row">
    <img src="/logo.png" id="logo" data-testid="logo" />
  </header>
  <slot />
  <footer class="col">
    <hr />
    <div class="row" style="justify-content: space-between;">
      <LanguageSelection />
      <small>Công ty Namco</small>
    </div>
  </footer>
</main>

<style>
  main {
    max-width: 70rem;
    margin: 0 auto;
    padding: 1rem 1rem 2rem;
  }

  @media only screen and (min-width: 58rem) {
    :root {
      font-size: 17px;
    }

    main {
      padding: 1rem 4rem 2rem;
    }
  }

  header {
    justify-content: center;
    flex-flow: nowrap;
  }

  #logo {
    object-fit: contain;
    background: none;
    height: 1rem;
  }

  footer {
    margin: 3rem 0 0;
  }
</style>
```

```html
<script lang="ts">
  import t from "$lib/localise";
  import vendors from "$lib/data/vendors";

  let vendorId: string;
</script>

<div class="col spread">
  <div class="col">
    <a href="/tours">{$t.tour}</a>
    <a href="/bookings">{$t.booking}</a>
    <a href="/book">{$t.createBooking}</a>
    <!-- <a href="/transactions">{$t.transaction}</a> -->
    <a href="/finance">{$t.finance}</a>
  </div>

  <select bind:value="{vendorId}">
    {#each vendors as { vendorId, name }}
    <option value="{vendorId}">{name}</option>
    {/each}
  </select>

  <div class="col">
    <a href="/serviceSummary/{vendorId}">{$t.serviceSummary}</a>
    <a href="/vendor/{vendorId}">{$t.vendor}</a>
  </div>
</div>
```

```css
hr {
  border: none;
  border-top: 1px solid var(--grey);
  margin: 0;
}

hr.break {
  border: none;
  margin: 0;
}

hr.hidden {
  border-color: var(--background);
}

button,
[type="button"] {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  place-content: center;
  text-decoration: none !important;
  height: var(--control-height);
  width: fit-content;
  padding: 0.7rem 0.9rem;
  border-radius: 2rem;
  touch-action: manipulation;
  cursor: pointer;
  color: var(--control-text);
  font-weight: 500;
  background: var(--control-background);
  border: none;
}

:where(button:enabled, [type="button"]):active {
  transform: scale(0.9);
  transition: transform 0.3s ease;
}

@media (hover: hover) {
  :where(button:enabled, [type="button"]):not(#google-pay button):not(
      .error
    ):not([class*="mapboxgl-ctrl"]):hover {
    color: var(--control-text-highlight);
    background: var(--control-background-highlight);
  }
}

:where(button:enabled, [type="button"]):not(#google-pay button):not(.error):not(
    [class*="mapboxgl-ctrl"]
  ):active {
  color: var(--control-text-highlight);
  background: var(--control-background-highlight);
}

/* Icon */

.icon {
  border-radius: 50%;
  width: var(--control-height);
}

/* Bold */

.bold {
  background: var(--bold-background);
  color: var(--bold-text);
  padding: 0 1rem;
  font-weight: 500;
}

:where(button:enabled, [type="button"]):not(.error).bold:hover {
  color: white !important;
  background: var(--pink) !important;
}

input[type="checkbox"],
input[type="radio"] {
  z-index: 100;
  position: relative;
  height: 1.25rem;
  width: 1.25rem;
  margin-right: 0.4rem;
  padding: 0;
  vertical-align: middle;
  cursor: pointer;
  border-radius: 0.3rem;
}

input[type="checkbox"]::after {
  content: "";
  box-sizing: border-box;
  opacity: 0;
  position: absolute;
  width: 0.3rem;
  height: 0.75rem;
  border: 2px solid var(--background);
  border-top: 0;
  border-left: 0;
  left: 0.4rem;
  top: 0.1rem;
  transform: rotate(35deg);
}

input[type="checkbox"]:checked::after {
  opacity: 1;
}

input[type="checkbox"]:checked,
input[type="radio"]:checked {
  border-color: var(--control-text-highlight);
  background: var(--control-text-highlight);
  color: var(--background);
}

/* rounded style */

.rounded {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  place-content: center;
  cursor: pointer;
  height: var(--control-height);
  width: fit-content;
  padding: 0 0.7rem;
  border-radius: var(--radius);
  border: 1px solid var(--grey);
  cursor: pointer;
}

input:where([type="checkbox"], [type="radio"]):has(+ .rounded) {
  display: none;
}

input:where([type="checkbox"], [type="radio"]) + .rounded {
  color: var(--control-text);
  border-width: var(--thick);
  border-color: var(--grey);
  padding: 0 0.8rem;
  border-radius: 2rem;
}

input:where([type="checkbox"], [type="radio"]):hover + .rounded {
  border-color: var(--input-highlight);
}

input:where([type="checkbox"], [type="radio"]):checked + .rounded {
  border-color: var(--control-text-highlight);
  border-width: var(--thick);
}

/* Group */

.checkboxes {
  display: flex;
  flex-flow: column wrap;
  row-gap: 0.5rem;
  column-gap: 1rem;
  overflow-x: auto;
}

.checkboxes label {
  flex-direction: row;
  align-items: center;
  row-gap: 1rem;
}

.frame {
  border: 1px solid var(--grey);
  border-radius: var(--radius);
  padding: 1rem;
  width: fit-content;
}

.fade-right {
  mask-image: linear-gradient(
    to left,
    rgba(225, 225, 225, 0),
    var(--background) 1.5rem
  );
}

.fade-bottom {
  mask-image: linear-gradient(
    to top,
    rgba(225, 225, 225, 0),
    var(--background) 2rem
  );
}

.message {
  display: flex;
  place-items: center;
  justify-content: center;
  text-align: center;
  gap: 0.5rem;
  font-weight: 450;
  padding: 0.5rem;
  color: var(--subtext);
}

:disabled,
.disabled {
  cursor: not-allowed !important;
  opacity: 0.3;
}

.error {
  border-color: orange !important;
  border-style: solid;
  border-width: var(--thick);
}

label,
[role="group"] {
  font-size: medium;
  font-weight: 500;
  color: var(--label);
  display: flex;
  flex-direction: column;
  row-gap: 0.3rem;
}

[role="group"] {
  row-gap: 0.5rem;
}

label {
  width: fit-content;
}

label > small:not(.suffix, .prefix) {
  margin-left: 0.5rem;
}

input,
select,
textarea,
label:has(> input[type="file"]),
label:has(.prefix, .suffix) {
  box-sizing: border-box;
  appearance: none;
  background: var(--input-background);
  border: var(--thick) solid var(--input-border);
  border-radius: var(--radius);
}

input,
label:has(.prefix, .suffix),
select {
  height: var(--control-height);
  width: fit-content;
  max-width: 100%;
  vertical-align: middle;
  display: flex;
  padding: 0 1rem;
  margin: 0;
  color: inherit;
  font-weight: 400;
}

label:has(.prefix, .suffix) {
  flex-direction: row;
  align-items: center;
  gap: 0.3rem;
  padding: 0 1rem;
}

label:has(.prefix, .suffix) > input {
  border: none;
  height: unset;
  padding: 0;
  margin: 0;
  border-radius: 0;
}

select {
  cursor: pointer;
}

textarea {
  font-weight: 400;
  padding: 0.7rem;
  width: 100%;
  max-width: 100%;
  font-size: 100%;
}

textarea::-webkit-resizer {
  border-width: 0.5rem;
  border-radius: 0 0 0.5rem 0;
  border-style: solid;
  border-color: transparent var(--control-background) var(--control-background)
    transparent;
}

label:has(> input[type="file"]) {
  width: 100%;
  height: 7rem;
  place-items: center;
  justify-content: space-evenly;
  border-style: dashed;
}

::placeholder {
  color: var(--placeholder);
  font-weight: inherit;
}

input[type="file"],
::file-selector-button {
  display: none;
}

label:hover:has(> input[type="file"]),
:where(select, textarea):focus,
label:not(:has(.prefix, .suffix)) > input:focus,
label:has(.prefix, .suffix):focus-within {
  border-color: var(--input-highlight);
  box-shadow: 0 0 0 1px var(--input-highlight);
  border-style: solid;
}

select:hover {
  color: var(--control-text);
}

.gallery {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 0.5rem;
}

.gallery:has(> iframe) {
  grid-template-columns: repeat(3, 1fr);
}

@media only screen and (max-width: 37rem) {
  .gallery {
    grid-template-columns: repeat(3, 1fr);
  }

  .gallery:has(> iframe) {
    grid-template-columns: repeat(2, 1fr);
  }
}

.gallery > .card {
  display: flex;
}

.gallery img {
  object-fit: cover;
  object-position: top;
  aspect-ratio: 3/4.5;
}

.gallery video {
  object-fit: contain;
  aspect-ratio: 16/9;
}

img,
video,
iframe {
  display: block;
  object-fit: cover;
  margin: 0;
  width: 100%;
  border-radius: var(--radius);
  background-color: var(--grey);
  background-image: linear-gradient(
    90deg,
    var(--grey),
    var(--background),
    var(--grey)
  );
  background-size: 300px 100%;
  background-repeat: no-repeat;
  animation: sweep 1.2s ease-in-out infinite;
  display: inline-block;
}

iframe {
  height: 100%;
  aspect-ratio: 16/9;
  overflow: hidden;
}

@keyframes sweep {
  0% {
    background-position-x: -300px;
  }
  100% {
    background-position-x: calc(300px + 100%);
  }
}

@import url("reset.css");
@import url("space.css");
@import url("basic.css");
@import url("button.css");
@import url("checkbox.css");
@import url("custom.css");
@import url("error.css");
@import url("form.css");
@import url("gallery.css");
@import url("image.css");
@import url("modal.css");
@import url("pills.css");
@import url("summary.css");
@import url("table.css");
@import url("typography.css");
@import url("light.css");
@import url("dark.css");

:root {
  --orange: #f38068;

  --thick: 1px;
  --radius: 0.6rem;

  --background: #faefee;
  --opaque: 255, 255, 255;
  --image-shadow: 0 7px 15px rgba(0, 0, 0, 0.2);

  color: var(--text);
  --text: #393953;
  --label: #393953;
  --subtext: #717171;
  --placeholder: #a9a9a9;
  --grey: #c7c1c1;

  --anchor: #ff746d;
  --anchor-highlight: black;

  --input-border: #c7c1c1;
  --input-background: #fef4f4;
  --input-highlight: var(--orange);

  --control-text: var(--text);
  --control-text-highlight: black;
  --control-background: #b2d5d9;
  --control-background-highlight: #abd0d5;
  --control-height: 2.3rem;

  --bold-text: white;
  --bold-background: var(--orange);
  --pink: #393953;

  --green: #6d9a66;
  --amber: #f7c777;
  --red: #c13e3e;
}

dialog[open] {
  max-width: 90%;
  min-width: 15rem;
  width: fit-content;
  height: fit-content;
  max-height: 90svh;
  padding: 1rem 1.5rem 1.5rem;
  overflow: auto;
  border: none;
  border-radius: var(--radius);
  animation: enter 0.3s ease;
  background: var(--background);
  border: var(--thick) solid var(--grey);
}

@media only screen and (min-width: 58rem) {
  :modal {
    padding: 2rem 2.5rem 2.5rem;
  }
}

dialog::backdrop {
  background: rgba(34, 34, 34, 0.4);
  animation: appear 0.3s ease;
}

@keyframes appear {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes enter {
  from {
    opacity: 0;
    transform: translateY(3rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.pills > div,
.pill {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  place-content: center;
  text-decoration: none !important;
  height: 2rem;
  width: fit-content;
  padding: 0 0.5rem;
  border-radius: var(--radius);
  font-weight: 400;
  background: var(--background);
  border: 1px solid var(--grey);
  color: var(--text);
}

.pills > a:active {
  color: var(--control-text-highlight);
  background: var(--control-background-highlight);
}

*:not([class*="mapboxgl-ctrl"]) {
  outline: none;
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background: var(--background);
  border-radius: 0;
  -webkit-tap-highlight-color: transparent;
  overflow-wrap: break-word;
  -webkit-tap-primary-focus-color: transparent;
}

input:not([type="checkbox"]),
select,
textarea {
  width: 100%;
}

button,
input {
  font-size: 100%;
  line-height: 1.15;
}

input,
select,
textarea {
  font: inherit;
}

b {
  font-weight: 600;
  font-size: inherit;
}

::-moz-focus-inner {
  border-style: none;
  padding: 0;
}

:-moz-ui-invalid {
  box-shadow: none;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  appearance: none;
  margin: 0;
}

video::-webkit-media-controls-panel {
  background-image: none !important;
  filter: invert(1);
}

img {
  clip-path: inset(1px);
}

input[type="number"] {
  appearance: textfield;
}

.row {
  display: flex;
  flex-flow: row wrap;
  gap: 0.5rem;
  align-items: center;
}

.col {
  display: flex;
  flex-flow: column nowrap;
  gap: 1rem;
}

.row.spread {
  column-gap: 1.5rem;
}

.col.spread {
  row-gap: 1.5rem;
}

summary {
  touch-action: manipulation;
  cursor: pointer;
  font-weight: 500;
  width: 100%;
  padding: 1rem 0;
  border-bottom: var(--thick) solid var(--grey);
  transition: margin 150ms ease-out;
  list-style: none;
}

summary::after {
  float: right;
  font-family: "Font Awesome 5 Pro";
  font-weight: 900;
  content: "\f054";
}

details[open] summary::after {
  transform: rotate(90deg);
}

summary::-webkit-details-marker {
  display: none;
}

summary:hover {
  color: var(--anchor-highlight);
}

details {
  width: 100%;
}

details[open] {
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--grey);
}
details[open] > :first-child {
  margin-bottom: 1rem;
}

table {
  /* display: block;; */
  overflow-x: auto;
  overflow-y: hidden;
  border-collapse: collapse;
  border-style: hidden;
  text-indent: 0;
  text-align: left;
  column-gap: 1rem;
}

thead {
  border-bottom: 1.5px solid #747487;
  padding-top: 1rem;
}

th {
  border: none;
}

th {
  font-weight: 550;
  color: var(--label);
}

td,
th {
  padding: 0.4rem 0.5rem;
}

td {
  border: 1px solid var(--grey);
}

tr:not(:first-child) {
  border-top: 1px solid var(--grey);
}

table * {
  margin: 0 !important;
}

.table-responsive {
  display: flex;
  flex-wrap: nowrap;
  overflow-x: auto;
}

.table-responsive th,
.table-responsive td {
  white-space: nowrap;
}

@font-face {
  font-family: "Inter";
  src: url("/Inter.ttf");
}

/* @font-face {
  font-family: "Noto Color Emoji";
  src: url("/NotoColorEmoji.ttf");
} */

:root {
  font-family: "Inter", sans-serif;
  font-weight: 400;
  font-size: 18px;
  line-height: 1.4;
}

h1,
h2,
h3,
h4 {
  font-weight: 550;
  margin: 0;
  padding: 0;
  line-height: 81%;
  display: inline-block;
  color: var(--text);
}

p,
ul {
  margin: 0;
  padding: 0;
  font-size: 16px;
  color: var(--text);
}

ul {
  list-style-position: inside;
  padding-left: 0;
}

a {
  color: var(--anchor);
  font-weight: 450;
  text-decoration: none;
  cursor: pointer;
}

a:hover {
  color: var(--anchor-highlight);
}

::selection {
  background: var(--placeholder);
}

small {
  font-size: 90%;
  color: var(--subtext);
  font-weight: 450;
}

b {
  font-weight: 550;
}
```

```html
<!DOCTYPE html>
<html lang="en" data-bs-theme="auto" class="h-100">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%sveltekit.assets%/favicon.png" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Namco Du lịch</title>
    <link href="%sveltekit.assets%/icons/css/all.min.css" rel="stylesheet" />
    %sveltekit.head%
  </head>

  <body>
    <div>%sveltekit.body%</div>
  </body>
</html>
```

```ts
import { describe, it, expect } from "vitest";

describe("sum test", () => {
  it("adds 1 + 2 to equal 3", () => {
    expect(1 + 2).toBe(3);
  });
});
```

```ts
import { expect, test } from "@playwright/test";

test.describe("Booking Page", () => {
  test("should be able to select a tour, add guest details, and proceed to vnpay sandbox", async ({
    page,
  }) => {
    await page.goto("/book");

    const createGuestSelector = (index: number) => {
      const baseSelector = `[data-testid="guest-element-${index}"]`;
      return {
        name: `${baseSelector} [data-testid="name"]`,
        whatsapp: `${baseSelector} [data-testid="whatsapp"]`,
        email: `${baseSelector} [data-testid="email"]`,
        vegetarian: `${baseSelector} [data-testid="vegetarian"]`,
        drivingChoice: `${baseSelector} [data-testid="driving-choice"]`,
      };
    };

    await page.selectOption("data-testid=tour-date-select", { index: 1 });

    const firstGuest = createGuestSelector(0);
    await page.fill(firstGuest.name, "John Doe");
    await page.fill(firstGuest.whatsapp, "+44 7783659838");
    await page.fill(firstGuest.email, "john.doe@example.com");
    await page.check(firstGuest.vegetarian);
    await page.selectOption(firstGuest.drivingChoice, "selfDrive");

    // await page.click("data-testid=add-guest-button");

    // const secondGuest = createGuestSelector(1);
    // await page.fill(secondGuest.name, "Jane Smith");
    // await page.fill(secondGuest.whatsapp, "+009876543210");
    // await page.fill(secondGuest.email, "jane@example.com");
    // await page.check(secondGuest.vegetarian);
    // await page.selectOption(secondGuest.drivingChoice, "easyRider");

    await page.fill("data-testid=passcode", "123");

    const invoiceTotal = await page.textContent(
      "data-testid=invoice-total-amount"
    );
    // expect(invoiceTotal).toContain("8.200.000");

    await page.click("data-testid=submit-button");

    // ! Uncomment this when payment gateway is ready
    // const regexPattern =
    //   /.*vnpayment\.vn\/paymentv2\/Transaction\/PaymentMethod\.html.*/;

    // await page.waitForURL(regexPattern, {
    //   waitUntil: "domcontentloaded",
    // });

    // const testCard = {
    //   bankCode: "NCB",
    //   cardNumber: "9704198526191432198",
    //   cardHolder: "NGUYEN VAN A",
    //   cardDate: "07/15",
    //   otp: "123456",
    // };

    // await page.waitForSelector("body > div.loading > .lds-ring", {
    //   state: "hidden",
    // });

    // await page.waitForSelector('[data-bs-target="#accordionList2"]');

    // await page.click('[data-bs-target="#accordionList2"]');

    // await page.click(`id=${testCard.bankCode}`);

    // await page.waitForSelector("body > div.loading > .lds-ring", {
    //   state: "hidden",
    // });

    // // Check if the span has the expected text
    // await expect(page.locator("#totalAmountDt")).toHaveText("8.200.000");

    // await page.fill("input#card_number_mask", testCard.cardNumber);
    // await page.fill("id=cardHolder", testCard.cardHolder);
    // await page.fill("id=cardDate", testCard.cardDate);

    // await page.click("id=btnContinue");

    // await page.waitForSelector("body > div.loading > .lds-ring", {
    //   state: "hidden",
    // });

    // await page.fill("id=otpvalue", testCard.otp);
    // await page.click("id=btnConfirm");

    // Wait for the confirmation page to load
    await page.waitForURL(/.*book\/confirm.*/);

    await expect(
      page.locator("data-testid=payment-confirmation-success-alert")
    ).toBeVisible();
  });
});
```

```ts
import { expect, test } from "@playwright/test";

test("home page has logo", async ({ page }) => {
  await page.goto("/");
  await expect(page.getByTestId("logo")).toBeVisible();
});
```

```text
.DS_Store
node_modules
/build
/.svelte-kit
/package
.env
.env.*
!.env.example
vite.config.js.timestamp-*
vite.config.ts.timestamp-*

/.vercel
proomt.md
test-results
```

```json
{
  "name": "namco-backoffice",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "test": "npm run test:integration && npm run test:unit",
    "check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
    "check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
    "test:integration": "playwright test --ui",
    "test:unit": "vitest"
  },
  "devDependencies": {
    "@playwright/test": "^1.39.0",
    "@sveltejs/adapter-auto": "^2.0.0",
    "@sveltejs/adapter-vercel": "^3.0.3",
    "@sveltejs/kit": "^1.20.4",
    "dotenv": "^16.3.1",
    "svelte": "^4.0.5",
    "svelte-check": "^3.4.3",
    "tslib": "^2.4.1",
    "typescript": "^5.0.0",
    "vite": "^4.4.2",
    "vitest": "^0.34.6",
    "case": "^1.6.3",
    "date-fns": "^2.30.0",
    "resend": "^1.1.0",
    "surrealdb.js": "0.5.0",
    "vnpay": "^0.2.0"
  },
  "type": "module"
}
```

```ts
import type { PlaywrightTestConfig } from "@playwright/test";

const configPreview: PlaywrightTestConfig = {
  webServer: {
    command: "npm run build && npm run preview",
    port: 4173,
  },
  testDir: "tests",
  testMatch: /(.+\.)?(test|spec)\.[jt]s/,
};

const configLocal: PlaywrightTestConfig = {
  webServer: {
    command: "npm run dev",
    url: "http://localhost:5173",
    reuseExistingServer: true,
  },
  testDir: "tests",
  testMatch: /(.+\.)?(test|spec)\.[jt]s/,
  use: {
    baseURL: "http://localhost:5173",
  },
};

export default configLocal;
```

```js
import vercel from "@sveltejs/adapter-vercel";
import { vitePreprocess } from "@sveltejs/kit/vite";

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),
  kit: {
    adapter: vercel(),
  },
  onwarn: (warning, handler) => {
    if (warning.code.startsWith("a11y-")) {
      return;
    }
    handler(warning);
  },
};

export default config;
```

```json
{
  "github": {
    "silent": true
  }
}
```

```ts
import { sveltekit } from "@sveltejs/kit/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [sveltekit()],
  test: {
    include: ["src/**/*.{test,spec}.{js,ts}"],
  },
});
```

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch server",
      "request": "launch",
      "runtimeArgs": ["run-script", "dev"],
      "runtimeExecutable": "npm",
      "skipFiles": ["<node_internals>/**"],
      "type": "node",
      "console": "integratedTerminal"
    },

    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://127.0.0.1:5173",
      "webRoot": "${workspaceFolder}"
    }
  ],
  "compounds": [
    {
      "name": "Both",
      "configurations": ["Launch server", "Launch Chrome"]
    }
  ]
}
```

```ts
import { SURREAL_URL } from "$env/static/private";
const { BASE_URL } = import.meta.env;
import Surreal from "surrealdb.js";

export default async function initDb() {
  const db = new Surreal(SURREAL_URL);

  await db.use("namco", database);
  return db;
}
```

```dockerfile
FROM zenika/alpine-chrome:with-puppeteer
WORKDIR /usr/src/app
USER chrome
COPY --chown=chrome:chrome package*.json ./
RUN npm install
COPY --chown=chrome:chrome . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```js
const express = require("express");
const fetch = require("node-fetch");
const { Client, MessageMedia, LocalAuth } = require("whatsapp-web.js");
const QRCode = require("qrcode");
const { sendTelegramPhotoMessage } = require("./telegram");

const app = express();

app.use(express.json());

const whatsapp = new Client({
  puppeteer: {
    headless: true,
    args: ["--no-sandbox"],
  },
  authStrategy: new LocalAuth(),
});

whatsapp.on("qr", (qr) => {
  QRCode.toBuffer(qr)
    .then((buffer) => {
      sendTelegramPhotoMessage(
        "Scan using Namco WhatsApp Business to signin ",
        buffer
      );
    })
    .catch((error) => {
      console.error("Error generating QR code:", error.message);
    });
});

whatsapp.on("ready", () => {
  console.log("WhatsApp client is ready!");
});

whatsapp.initialize();

function formatWhatsAppId(phoneNumber) {
  let formattedNumber = phoneNumber.replace(/[^\d]/g, "");

  let whatsappId = formattedNumber + "@c.us";

  return whatsappId;
}

app.get("/alive", (req, res) => {
  res.status(200).json({ status: "success", message: "API is alive!" });
});

app.post("/create-group", async (req, res) => {
  const { groupName, imageUrl, participants } = req.body;
  const group = await whatsapp.createGroup(
    groupName,
    participants.map((number) => formatWhatsAppId(number))
  );

  const groupChat = await whatsapp.getChatById(group.gid._serialized);

  await groupChat.setInfoAdminsOnly(true);

  const response = await fetch(imageUrl);
  const buffer = await response.buffer();
  const groupPicture = new MessageMedia("image/jpg", buffer.toString("base64"));
  await groupChat.setPicture(groupPicture);

  const inviteCode = await groupChat.getInviteCode();

  res.status(200).json({
    status: "success",
    message: "Group created!",
    groupId: group.gid._serialized,
    inviteCode,
  });
});

app.post("/add-guest-to-group", async (req, res) => {
  const { groupId, participants } = req.body;
  const chat = await whatsapp.getChatById(groupId);
  if (!chat.isGroup) {
    res.status(400).json({
      status: "failure",
      message: "Provided chat ID does not belong to a group!",
    });
    return;
  }
  await chat.addParticipants(
    participants.map((number) => formatWhatsAppId(number))
  );
  res.status(200).json({ status: "success", message: "Users added!" });
});

app.post("/send-message-to-group", async (req, res) => {
  const { groupId, message } = req.body;
  const chat = await whatsapp.getChatById(groupId);
  if (!chat.isGroup) {
    res.status(400).json({
      status: "failure",
      message: "Provided chat ID does not belong to a group!",
    });
    return;
  }
  await chat.sendMessage(message);
  res.status(200).json({ status: "success", message: "Message sent!" });
});

app.listen(3000, () => {
  console.log("WhatsApp client will be ready soon...");
});
```

```js
const axios = require("axios");
const FormData = require("form-data");
require("dotenv").config();

async function sendTelegramTextMessage(text) {
  try {
    await axios.post(
      `https://api.telegram.org/bot${process.env.TELEGRAM_TOKEN}/sendMessage`,
      {
        chat_id: process.env.TELEGRAM_CHAT_ID,
        text,
      },
      {
        headers: {
          Accept: "application/json",
          "Content-Type": "application/json",
        },
      }
    );
  } catch (error) {
    console.error("Error sending Telegram text:", error.message);
  }
}

async function sendTelegramPhotoMessage(text, imageBuffer) {
  const formData = new FormData();
  formData.append("chat_id", process.env.TELEGRAM_CHAT_ID);
  formData.append("photo", imageBuffer, "qrcode.png");
  formData.append("caption", text);

  try {
    await axios.post(
      `https://api.telegram.org/bot${process.env.TELEGRAM_TOKEN}/sendPhoto`,
      formData,
      {
        headers: formData.getHeaders(),
      }
    );
  } catch (error) {
    console.error("Error sending Telegram photo:", error.message);
  }
}

module.exports = {
  sendTelegramTextMessage,
  sendTelegramPhotoMessage,
};
```
