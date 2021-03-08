<script>
  // components
  import Navbar from "./components/Layout/Navbar.svelte";
  import ExpenseList from "./components/Expenses/ExpenseList.svelte";
  import Totals from "./components/Expenses/Totals.svelte";
  import ExpenseForm from "./components/Expenses/ExpenseForm.svelte";
  import Footer from "./components/Layout/Footer.svelte";
  // modal
  import Modal from "./components/Shared/Modal.svelte";
  //  general imports
  import { setContext, onMount, afterUpdate } from "svelte";

  let expenses = [];
  // set editing variables
  let setName = "";
  let setAmount = null;
  let setId = null;
  $: isEditing = setId ? true : false;
  // toggle form
  let isFormOpen = false;
  function showForm() {
    isFormOpen = true;
  }
  function hideForm() {
    isFormOpen = false;
    setName = "";
    setAmount = null;
    setId = null;
  }
  // remove expense
  function removeExpense(id) {
    expenses = expenses.filter((item) => item.id !== id);
  }
  setContext("remove", removeExpense);
  setContext("modify", setModifiedExpense);
  // reactive
  $: total = expenses.reduce((acc, curr) => {
    return (acc += curr.amount);
  }, 0);
  //add expense
  function addExpense({ name, amount }) {
    let expense = { id: Math.random(), name, amount };
    expenses = [expense, ...expenses];
  }
  // setModifiedExpense
  function setModifiedExpense(id) {
    let expense = expenses.find((item) => item.id === id);
    setId = expense.id;
    setName = expense.name;
    setAmount = expense.amount;
    // showForm
    showForm();
  }
  // editExpense
  function editExpense({ name, amount }) {
    expenses = expenses.map((item) => {
      return item.id === setId ? { ...item, name, amount } : { ...item };
    });
    setId = null;
    setAmount = null;
    setName = "";
  }
  // clear Expenses
  function clearExpenses() {
    expenses = [];
  }
  function setLocalStorage() {
    localStorage.setItem("expenses", JSON.stringify(expenses));
  }
  onMount(() => {
    expenses = localStorage.getItem("expenses")
      ? JSON.parse(localStorage.getItem("expenses"))
      : [];
  });
  afterUpdate(() => {
    setLocalStorage();
  });
</script>

<Navbar title="Budget Calculator" {showForm} />
<main class="content">
  {#if isFormOpen}
    <Modal>
      <ExpenseForm
        bind:name={setName}
        bind:amount={setAmount}
        {addExpense}
        {editExpense}
        {isEditing}
        {hideForm}
      />
    </Modal>
  {/if}
  <Totals title="total expenses" {total} />
  <ExpenseList {expenses} />
  <button
    type="button"
    class="btn btn-primary btn-block"
    on:click={clearExpenses}
  >
    clear expenses
  </button>
</main>
<Footer />
