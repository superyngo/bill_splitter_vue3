<script lang="ts">
import {defineComponent, ref} from "vue";
import type {Ref} from "vue";

interface item {
  id: string;
  name: string;
  cost: number;
  paidMemberId: string;
}

interface items {
  items: item[];
}

interface member {
  id: string;
  name: string;
  paidAmount: number;
  ownedAmount: number;
}

interface members {
  members: member[];
}

type saveData = {
  items: items;
  members: members;
};

export default defineComponent({
  data() {
    return {
      items: Array<{
        id: string;
        name: string;
        cost: number;
        paidMemberId: string;
      }>(),
      newItem: {
        id: "",
        name: "",
        cost: null,
        paidMemberId: "whom?",
      },
      members: Array<{
        id: string;
        name: string;
        paidAmount: number;
        ownedAmount: number;
      }>(),
      newMember: {
        id: "",
        name: "",
        paidAmount: 0,
        ownedAmount: 0,
      },
      settlement: {
        totalPaid: 0,
        notYetPaid: 0,
        averagePay: 0,
        totalMembers: 0,
      },
      currencyFormat: new Intl.NumberFormat("en-US", {
        currency: "USD",
        style: "currency",
        minimumFractionDigits: 0,
      }),
    };
  },
  methods: {
    addItem(): void {
      if (typeof this.newItem.cost == "number" && !isNaN(this.newItem.cost)) {
        this.newItem.paidMemberId =
          this.newItem.paidMemberId === "whom?"
            ? "uncertain"
            : this.newItem.paidMemberId;
        this.newItem.id = crypto.randomUUID();
        console.log(this.newItem.name);
        this.newItem.name = this.newItem.name || "無品名";
        this.items.push({...this.newItem});
        this.newItem = {
          id: "",
          name: "",
          cost: null,
          paidMemberId: "whom?",
        };
      } else {
        alert("請以正確數字輸入價錢");
      }
    },
    removeItem(index: number): void {
      this.items.splice(index, 1);
    },
    addMember(): void {
      if (this.newMember.name == "") {
        alert("請輸入人名");
      } else {
        this.newMember.id = crypto.randomUUID();
        this.members.push({...this.newMember});
        this.newMember.name = "";
      }
    },
    removeMember(index: number): void {
      this.items.forEach((item) => {
        item.paidMemberId =
          item.paidMemberId === this.members[index].id
            ? "uncertain"
            : item.paidMemberId;
      });
      this.newItem.paidMemberId =
        this.newItem.paidMemberId === this.members[index].id
          ? "uncertain"
          : this.newItem.paidMemberId;
      this.members.splice(index, 1);
    },
    calculateCost(): void {
      this.settlement.totalPaid = this.total();
      this.settlement.totalMembers = this.members.length;
      this.settlement.averagePay =
        this.settlement.totalMembers === 0
          ? 0
          : Math.floor(
              this.settlement.totalPaid / this.settlement.totalMembers
            );
      this.members.forEach((member) => {
        member.paidAmount = this.items
          .filter((item: item) => item.paidMemberId === member.id)
          .reduce((acc: number, item: item) => acc + item.cost, 0);
        member.ownedAmount = member.paidAmount - this.settlement.averagePay;
      });
    },
    saveData(): void {
      const saveData: saveData = {
        items: this.items,
        members: this.members,
      };
      const saveDataString = JSON.stringify();
      localStorage.setItem("saveData", saveData);
    },
    total(): number {
      const total: number = this.items.reduce(
        (acc: number, item: item) => acc + item.cost,
        0
      );
      return total;
    },
    close(): void {
      this.items = Array<{
        id: string;
        name: string;
        cost: number;
        paidMemberId: string;
      }>();
      this.members = Array<{
        id: string;
        name: string;
        paidAmount: number;
        ownedAmount: number;
      }>();
      this.calculateCost();
    },
  },
  watch: {
    items: {
      deep: true,
      handler(): void {
        this.calculateCost();
        this.saveData();
      },
    },
    members: {
      deep: true,
      handler(): void {
        this.calculateCost();
        this.saveData();
      },
    },
  },
  computed: {},
  mounted() {
    const saveData = JSON.parse(localStorage.getItem("saveData"));
    this.items = saveData.items ?? this.items;
    this.members = saveData.members ?? this.members;
    this.calculateCost();
  },
});
</script>

<template>
  <div class="wrapper">
    <h1>攤價計算器</h1>
    <div class="inputWrap">
      <div class="members grid">
        <label class="newMember">
          <input
            type="text"
            class="MemberNameInput"
            v-model="newMember.name"
            placeholder="輸入人名"
          />
          <button class="addMember" @click="addMember">
            <img
              src="/src/user.png"
              title="user icons User icons created by Amazona Adorada - Flaticon"
            />
          </button>
        </label>

        <ul class="membersList">
          <p v-if="members.length === 0">尚無攤價人頭</p>
          <li
            class="member"
            v-for="(member, index) of members"
            :key="member.id"
          >
            <span>{{ member.name }} </span>
            <div class="information">
              <span
                >已支付：{{ currencyFormat.format(member.paidAmount) }}</span
              >

              <span v-if="member.ownedAmount >= 0" class="green">
                應退還 ：{{
                  currencyFormat.format(Math.abs(member.ownedAmount))
                }}
              </span>
              <span v-if="member.ownedAmount < 0" class="red">
                應收取 ：{{
                  currencyFormat.format(Math.abs(member.ownedAmount))
                }}
              </span>
              <button class="removeMember" @click="removeMember(index)">
                X
              </button>
            </div>
          </li>
        </ul>
      </div>
      <hr />
      <div class="items grid">
        <label class="newItem">
          <input
            type="text"
            class="itemNameInput"
            placeholder="買什麼？"
            v-model="newItem.name"
          />
          <input
            type="number"
            class="itemCostInput"
            placeholder="多少錢？"
            v-model="newItem.cost"
          />
          <select class="itemPaidByWhom" v-model="newItem.paidMemberId">
            <option value="whom?" disabled selected>誰付？</option>
            <option value="uncertain">待付</option>
            <option v-for="member of members" :value="member.id">
              {{ member.name }}
            </option>
          </select>
          <button class="addItem" @click="addItem">
            <img src="/src/shopping-bag.png" />
          </button>
        </label>

        <ul class="itemsList">
          <p v-if="items.length === 0">尚無支出款項</p>

          <li class="itemList" v-for="(item, index) of items" :Key="item.id">
            <span>{{ items[index].name }}</span>
            <div class="information">
              <span>價錢： {{ currencyFormat.format(items[index].cost) }}</span>
              <span
                >付款人：
                <select v-model="item.paidMemberId">
                  <option value="uncertain">待付</option>
                  <option v-for="member of members" :value="member.id">
                    {{ member.name }}
                  </option>
                </select>
              </span>
              <button class="removeItem" @click="removeItem(index)">X</button>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <hr />

    <div class="settlement">
      <div>
        <div>
          <span>總花費：{{ currencyFormat.format(settlement.totalPaid) }}</span>
          <span>總人數：{{ settlement.totalMembers }}</span>
          <span>
            人均支出：{{ currencyFormat.format(settlement.averagePay) }}
          </span>
        </div>
        <br />
        <button @click="close">結束攤價</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  background: rgba(255, 255, 255, 0.701);
  border: 1px solid black;
  border-radius: 1rem;
  width: 28.125rem;
}
h1 {
  text-align: center;
  margin: 1rem 0 0 0;
}
.inputWrap {
  display: grid;
}
.settlement {
  grid-column: 1 / span2;
}
hr {
  border: 1px solid black;
}
input,
select {
  width: 4rem;
  height: 1.5rem;
  margin: 0 0.5rem;
}
label {
  display: flex;
  justify-content: center;
  margin: 1rem 0;
}
p {
  text-align: center;
}

li {
  /* list-style: decimal; */
  display: flex;
  justify-content: space-evenly;
  margin: 0.5rem 0;
}

button {
  cursor: pointer;
  border: none;
  background-color: transparent;
}

img {
  width: 2rem;
}

img:hover {
  scale: 1.3;
  rotate: 10deg;
}
.settlement > * {
  margin: 1rem;
}

.settlement > div {
  display: grid;
  place-content: center;
  gap: 1rem;
}
.settlement > div > div {
  display: flex;
  gap: 5rem;
}
.red {
  color: red;
}
.green {
  color: green;
}
</style>
