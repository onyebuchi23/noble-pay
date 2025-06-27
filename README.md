<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cash Pay</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: "Inter", sans-serif;
            background-color: #f0f2f5; /* Light gray background */
        }
        /* Custom spinner animation */
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        /* Spinner for initial payment and bank transfer verification */
        .spinner-purple {
            border-top: 4px solid #8B5CF6; /* Purple color */
        }
        /* Spinner for confirming payment */
        .spinner-orange {
            border-top: 4px solid #F97316; /* Orange-500 color */
        }

        /* Base style for progress bar filler */
        .progress-bar-fill {
            height: 100%;
            border-radius: inherit;
            width: 0%; /* Start at 0% */
            transition: width linear; /* Smooth transition for width change */
        }

        /* Progress bar for initial payment */
        .progress-bar-purple {
            background-color: #8B5CF6; /* Purple color */
        }
        /* Progress bar for confirming payment */
        .progress-bar-orange {
            background-color: #F97316; /* Orange-500 */
        }

        /* Style for error message */
        .error-message {
            color: #dc2626; /* Red-600 for errors */
            font-size: 0.875rem; /* text-sm */
            margin-top: 0.5rem; /* mt-2 */
        }
        /* Modal for messages (retained for general notifications like copy status) */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-content {
            background-color: white;
            padding: 2rem;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            text-align: center;
            max-width: 90%;
            width: 400px;
        }
        .modal-button {
            background-color: #4F46E5; /* Indigo-600 */
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.375rem;
            margin-top: 1rem;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        .modal-button:hover {
            background-color: #4338CA; /* Indigo-700 */
        }
        /* Custom styling for the red X icon on transaction failed screen */
        .failed-icon-circle {
            background-color: #ef4444; /* Red-500 from screenshot */
            border-radius: 50%;
            width: 60px; /* Adjusted size to match screenshot */
            height: 60px; /* Adjusted size to match screenshot */
            display: flex;
            align-items: center;
            justify-content: center;
            margin: 0 auto 1.5rem auto;
        }
        .failed-icon-x {
            color: white; /* White 'X' as in screenshot */
            font-size: 2.5rem; /* Adjusted size to match screenshot */
            font-weight: bold;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-indigo-900 text-white p-4 shadow-md" id="mainHeader">
        <div class="container mx-auto px-4">
            <h1 class="text-xl font-bold rounded" id="headerTitle">Cash Pay</h1>
        </div>
    </header>

    <!-- Main Content (Payment Form) -->
    <main id="paymentForm" class="flex-grow flex items-center justify-center p-4">
        <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-sm sm:max-w-md md:max-w-lg lg:max-w-xl">
            <div class="mb-4">
                <label for="amount" class="block text-gray-700 text-sm font-medium mb-2 rounded">Amount</label>
                <input type="text" id="amount" class="shadow-sm appearance-none border rounded-lg w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent bg-gray-100 cursor-not-allowed" value="₦5,500" readonly>
            </div>
            <div class="mb-4">
                <label for="fullName" class="block text-gray-700 text-sm font-medium mb-2 rounded">Full Name</label>
                <input type="text" id="fullName" class="shadow-sm appearance-none border rounded-lg w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent" placeholder="Your full name">
                <p id="fullNameError" class="error-message hidden">Full Name is required.</p>
            </div>
            <div class="mb-6">
                <label for="emailAddress" class="block text-gray-700 text-sm font-medium mb-2 rounded">Your Email Address</label>
                <input type="email" id="emailAddress" class="shadow-sm appearance-none border rounded-lg w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent" placeholder="email address">
                <p id="emailAddressError" class="error-message hidden">Valid Email Address is required.</p>
            </div>
            <button id="payButton" type="submit" class="w-full bg-indigo-900 hover:bg-indigo-800 text-white font-bold py-3 px-4 rounded-lg focus:outline-none focus:shadow-outline transition duration-300 ease-in-out">
                Pay
            </button>
        </div>
    </main>

    <!-- Loading Screen (Initial Payment) -->
    <div id="loadingScreen" class="fixed inset-0 bg-white flex flex-col items-center justify-center hidden">
        <div class="spinner spinner-purple mb-6"></div>
        <h2 class="text-2xl font-bold text-gray-800 mb-2">Preparing Payment Account</h2>
        <p class="text-gray-600 text-center mb-6 px-4">Please wait while we set up your payment...</p>
        <div class="w-64 bg-gray-200 rounded-full h-2.5">
            <div id="progressBarPurple" class="progress-bar-fill progress-bar-purple"></div>
        </div>
    </div>

    <!-- Payment Notice Screen -->
    <div id="paymentNoticeScreen" class="fixed inset-0 bg-indigo-900 flex flex-col items-center justify-center hidden p-4">
        <div class="bg-orange-50 rounded-lg shadow-lg p-6 w-full max-w-sm sm:max-w-md md:max-w-lg lg:max-w-xl text-center">
            <h2 class="text-xl sm:text-2xl font-bold text-orange-600 mb-4 rounded-md p-2">Important payment notice:</h2>
            <p class="text-gray-800 mb-6 text-sm sm:text-base leading-relaxed">
                Please do not use Opay to make your payment. Our company does not authorize Opay for direct
                transfers from our platform. If you make a payment using Opay, your
                payment may not be confirmed.
            </p>
            <h3 class="text-lg sm:text-xl font-bold text-gray-800 mb-4">If you are using Opay:</h3>
            <ol class="list-decimal list-inside text-gray-800 text-left mb-6 space-y-2 text-sm sm:text-base px-4">
                <li>Transfer the money to Anthem bank account.</li>
                <li>Then use that bank account to complete your transaction.</li>
            </ol>
            <button id="okButton" class="bg-orange-500 hover:bg-orange-600 text-white font-bold py-3 px-6 rounded-lg focus:outline-none focus:shadow-outline transition duration-300 ease-in-out w-full sm:w-auto">
                OK
            </button>
        </div>
    </div>

    <!-- Bank Transfer Screen -->
    <div id="bankTransferScreen" class="fixed inset-0 bg-gray-100 hidden flex flex-col">
        <!-- Header for Bank Transfer Screen -->
        <header class="bg-white text-gray-800 p-4 shadow-md">
            <div class="container mx-auto px-4 flex justify-between items-center">
                <h1 class="text-xl font-bold rounded">Bank Transfer</h1>
                <button id="cancelButtonTransfer" class="text-red-500 font-semibold text-sm">Cancel</button>
            </div>
        </header>

        <main class="flex-grow flex flex-col items-center p-4 overflow-y-auto">
            <div class="w-full max-w-sm sm:max-w-md md:max-w-lg lg:max-w-xl bg-white shadow-lg rounded-lg p-6 mb-6 text-center">
                <div class="flex justify-between items-center mb-4">
                    <span class="text-lg font-bold text-gray-800">NGN 5,500</span>
                    <span id="displayEmailBankTransfer" class="text-sm text-gray-600"></span>
                </div>
                <p class="text-red-600 font-medium mb-4 text-left">
                    Please note: we do not accept any payment from opay bank
                </p>
                <div class="border border-gray-300 rounded-lg p-4 text-left">
                    <div class="flex justify-between items-center mb-3">
                        <div>
                            <p class="text-gray-600 text-sm">Amount</p>
                            <p id="transferAmount" class="font-bold text-gray-800">NGN 5,500</p>
                        </div>
                        <button class="bg-orange-400 hover:bg-orange-500 text-white text-xs font-semibold py-2 px-3 rounded-md transition duration-300 ease-in-out" onclick="copyToClipboard('NGN 5,500')">Copy</button>
                    </div>
                    <div class="flex justify-between items-center mb-3">
                        <div>
                            <p class="text-gray-600 text-sm">Account Number</p>
                            <p id="accountNumber" class="font-bold text-gray-800">1782299316</p>
                        </div>
                        <button class="bg-orange-400 hover:bg-orange-500 text-white text-xs font-semibold py-2 px-3 rounded-md transition duration-300 ease-in-out" onclick="copyToClipboard('8060695932')">Copy</button>
                    </div>
                    <div class="mb-3">
                        <p class="text-gray-600 text-sm">Bank Name</p>
                        <p class="font-bold text-gray-800">Access Bank</p>
                    </div>
                    <div>
                        <p class="text-gray-600 text-sm">Account Name</p>
                        <p class="font-bold text-gray-800">ONYEBUCHI JOSHUA</p>
                    </div>
                </div>
            </div>

            <div class="w-full max-w-sm sm:max-w-md md:max-w-lg lg:max-w-xl bg-white shadow-lg rounded-lg p-6 text-center">
                <p class="text-gray-700 mb-6">Pay to this specific account and get your access code</p>
                <button id="iHaveTransferredButton" class="bg-orange-500 hover:bg-orange-600 text-white font-bold py-3 px-6 rounded-lg focus:outline-none focus:shadow-outline transition duration-300 ease-in-out w-full">
                    I have made this bank Transfer
                </button>
            </div>
        </main>
    </div>

    <!-- Confirming Payment Loading Screen -->
    <div id="verifyPaymentLoadingScreen" class="fixed inset-0 bg-white flex flex-col items-center justify-center hidden">
        <div class="spinner spinner-orange mb-6"></div>
        <h2 class="text-2xl font-bold text-gray-800 mb-2">Confirming Your Payment</h2>
        <p class="text-gray-600 text-center mb-6 px-4">Please wait while we verify your transaction...</p>
        <div class="w-64 bg-gray-200 rounded-full h-2.5 mb-6">
            <div id="progressBarOrange" class="progress-bar-fill progress-bar-orange"></div>
        </div>
        <p class="text-gray-600 text-center text-sm px-4">This may take a few moments<br>Please do not close this page</p>
    </div>

    <!-- Transaction Not Successful Screen -->
    <div id="transactionFailedScreen" class="fixed inset-0 bg-gray-100 hidden flex flex-col">
        <!-- Header for Transaction Failed Screen -->
        <header class="bg-white text-gray-800 p-4 shadow-md">
            <div class="container mx-auto px-4 flex justify-between items-center">
                <h1 class="text-xl font-bold rounded">Bank Transfer</h1>
                <button id="cancelButtonFailed" class="text-red-500 font-semibold text-sm">Cancel</button>
            </div>
        </header>

        <main class="flex-grow flex flex-col items-center justify-center p-4">
            <div class="w-full max-w-sm sm:max-w-md md:max-w-lg lg:max-w-xl bg-white shadow-lg rounded-lg p-6 text-center">
                <div class="flex justify-between items-center mb-4">
                    <span class="text-lg font-bold text-gray-800">NGN 5,500</span>
                    <span id="displayEmailFailed" class="text-sm text-gray-600"></span>
                </div>
                <p class="text-red-600 font-medium mb-8 text-left">
                    Please note: we do not accept any payment from opay bank
                </p>

                <div class="failed-icon-circle mb-4">
                    <!-- Updated SVG for the red X icon -->
                    <svg class="failed-icon-x" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                        <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd"></path>
                    </svg>
                </div>
                <h2 class="text-xl sm:text-2xl font-bold text-orange-600 mb-2">Transaction Not Successful</h2>
                <p class="text-gray-700 mb-6 text-sm sm:text-base leading-relaxed">
                    Your payment Has Not Been Received Please Check The Details And Try Again
                </p>
                <button id="contactSupportButton" class="w-full bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-3 px-4 rounded-lg focus:outline-none focus:shadow-outline transition duration-300 ease-in-out flex items-center justify-center">
                    Contact Support
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-5 h-5 ml-2">
                        <path fill-rule="evenodd" d="M11.47 2.47a.75.75 0 0 1 1.06 0l7.5 7.5a.75.75 0 1 1-1.06 1.06L12 4.06 5.53 10.53a.75.75 0 0 1-1.06-1.06l7.5-7.5Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M12 2.25a.75.75 0 0 1 .75.75v9a.75.75 0 0 1-1.5 0V3a.75.75 0 0 1 .75-.75Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M12 12.25a.75.75 0 0 1-.75.75H3a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H12Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M21 12.25a.75.75 0 0 1-.75.75H12a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H21a.75.75 0 0 1 .75.75V12a.75.75 0 0 1-.75.75Z" clip-rule="evenodd" />
                        <path d="M12 12.25a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H21a.75.75 0 0 1 .75.75V12a.75.75 0 0 1-.75.75H12Z" />
                        <path fill-rule="evenodd" d="M12 12.25a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H21a.75.75 0 0 1 .75.75V12a.75.75 0 0 1-.75.75H12Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M12 12.25a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H3a.75.75 0 0 1-.75-.75V12a.75.75 0 0 1 .75-.75H12Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M12 12.25a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H21a.75.75 0 0 1 .75.75V12a.75.75 0 0 1-.75.75H12Z" clip-rule="evenodd" />
                        <path fill-rule="evenodd" d="M12 12.25a.75.75 0 0 1-.75-.75V6a.75.75 0 0 1 .75-.75H3a.75.75 0 0 1-.75-.75V12a.75.75 0 0 1 .75-.75H12Z" clip-rule="evenodd" />
                    </svg>
                </button>
            </div>
        </main>
    </div>


    <!-- Custom Modal for general messages (errors, success) -->
    <div id="messageModal" class="modal-overlay hidden">
        <div class="modal-content">
            <h3 id="modalTitle" class="text-xl font-bold mb-4"></h3>
            <p id="modalMessage" class="text-gray-700 mb-4"></p>
            <button id="modalCloseButton" class="modal-button">OK</button>
        </div>
    </div>

    <script>
        // Global variable to store user's email
        let userEmail = '';

        // Get references to elements
        const payButton = document.getElementById('payButton');
        const paymentForm = document.getElementById('paymentForm');
        const loadingScreen = document.getElementById('loadingScreen');
        const paymentNoticeScreen = document.getElementById('paymentNoticeScreen');
        const okButton = document.getElementById('okButton');
        const bankTransferScreen = document.getElementById('bankTransferScreen');
        const iHaveTransferredButton = document.getElementById('iHaveTransferredButton');
        const verifyPaymentLoadingScreen = document.getElementById('verifyPaymentLoadingScreen');
        const transactionFailedScreen = document.getElementById('transactionFailedScreen');
        const contactSupportButton = document.getElementById('contactSupportButton');
        const headerTitle = document.getElementById('headerTitle');
        const mainHeader = document.getElementById('mainHeader');
        const cancelButtonTransfer = document.getElementById('cancelButtonTransfer');
        const cancelButtonFailed = document.getElementById('cancelButtonFailed');

        const fullNameInput = document.getElementById('fullName');
        const emailAddressInput = document.getElementById('emailAddress');
        const fullNameError = document.getElementById('fullNameError');
        const emailAddressError = document.getElementById('emailAddressError');

        // Progress bar elements
        const progressBarPurple = document.getElementById('progressBarPurple');
        const progressBarOrange = document.getElementById('progressBarOrange');

        // Elements to display user email
        const displayEmailBankTransfer = document.getElementById('displayEmailBankTransfer');
        const displayEmailFailed = document.getElementById('displayEmailFailed');

        // Modal elements
        const messageModal = document.getElementById('messageModal');
        const modalTitle = document.getElementById('modalTitle');
        const modalMessage = document.getElementById('modalMessage');
        const modalCloseButton = document.getElementById('modalCloseButton');

        // Function to show custom modal (still used for copy feedback)
        function showModal(title, message) {
            modalTitle.textContent = title;
            modalMessage.textContent = message;
            messageModal.classList.remove('hidden');
        }

        // Function to hide custom modal
        function hideModal() {
            messageModal.classList.add('hidden');
        }

        // Event listener for modal close button
        modalCloseButton.addEventListener('click', hideModal);

        // Function to copy text to clipboard
        function copyToClipboard(text) {
            const textarea = document.createElement('textarea');
            textarea.value = text;
            document.body.appendChild(textarea);
            textarea.select();
            try {
                document.execCommand('copy');
                showModal('Copied!', 'The text has been copied to your clipboard.');
            } catch (err) {
                showModal('Error', 'Failed to copy text.');
            }
            document.body.removeChild(textarea);
        }

        // Function to r
