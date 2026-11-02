# marksold-market-place
A secure Consumer-to-Consumer (C2C) e-commerce marketplace built on Adalo featuring mandatory identity verification via Veriff and tiered seller subscriptions via Paystack.

🚀 Marksold MarketplaceShort Tagline: 
📋 Table of Contents📌 
1. Product Requirements Document (PRD)🗄️
2. Database Schema & Entity Architecture (Adalo Collections)🏗️
3. System Architecture & Adalo Data Flow🛡️
4. Security & Access Control🔌
5. API Contract Specifications🚀 Setup & Configuration📌

1. Product Requirements Document (PRD)❓ The ProblemHigh Scam & Theft Rates: Anonymous accounts on traditional C2C platforms lead to rampant fraudulent listings, stolen property sales, and untrustworthy buyers/sellers.Lack of Identity Oversight: Absence of mandatory ID checks creates unsafe peer-to-peer interactions with zero accountability.Unregulated Seller Onboarding: Failure to monetize and verify commercial users results in spam listings and degraded platform trust.💡 The SolutionA verified C2C e-commerce marketplace built on Adalo. Every user (buyer and seller) must undergo real-time biometric and government ID verification via Veriff upon registration before accessing marketplace listings. Sellers are routed through Paystack to purchase and renew business subscriptions before publishing multi-media listings.🎯 Key Functional Requirements🔐 FR-1 [User Management & Onboarding]: Register users with Buyer/Seller role selection, secure authentication, and profile tracking.🪪 FR-2 [Mandatory Veriff ID Verification]: Integrated external webview workflow requiring all registered buyers and sellers to complete government ID verification.💳 FR-3 [Paystack Subscription & Renewal]: Enforce seller subscription purchases and recurring renewals via Paystack checkout prior to listing creation.📦 FR-4 [Listing & Multi-Media Management]: Full CRUD support for product listings, supporting attached images, videos, and thumbnails.💬 FR-5 [Contact Seller & Chat System]: Integrated in-app messaging allowing verified buyers to initiate direct chatrooms with verified sellers.🗄️ 2. Database Schema & Entity Architecture (Adalo Collections)The backend runs on Adalo's Relational Database Collections, establishing clear foreign-key linkages between users, verifications, subscriptions, and marketplace inventory.📊 Entity-Relationship OverviewCollection NamePrimary Key / IdentifierRelational LinksKey Attributes & Description👤 UsersIDUser Types, VerificationsEmail, Password, First Name, Last Name, Verification Status, Is_Subscribed, Subscription End Date🪪 VerificationsIDUsers (User Email)Session ID, Process Status, Verification Status, Date Created, Last Updated🏷️ ListingsIDUsers (Seller ID), CategoriesListing Title, Listing Description, Location, Selling Price, Thumbnail Image, Listing Date🖼️ Listing Images / VideosIDListings (Listing ID)Image/Video Name, Image/Video Path, Image/Video Number💳 PaymentsIDUsers (User Email)Amount, Payment Date, Status, Reference💬 Chatrooms & MessagesIDUsers (Buyer ID, Seller ID)Chatroom Title, Text, Message Files, Message Images, Message Videos🏗️ 3. System Architecture & Adalo Data FlowThe architecture integrates Adalo's frontend client with external API engines and webhook handlers for real-time verification and transaction lifecycle management.┌──────────────────────────────────────────────────────────────────┐

│                    ADALO FRONTEND CLIENT APP                     │
│                                                                  │
│  ┌───────────────────────┐             ┌───────────────────────┐ │
│  │ Registration & Auth   │             │ Marketplace Feed &    │ │
│  │ (Buyer / Seller Role) │             │ In-App Chatrooms      │ │
│  └───────────┬───────────┘             └───────────────────────┘ │
└──────────────│───────────────────────────────────────────────────┘
               │
               │ 1. Mandatory Identity Redirect
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                   VERIFF IDENTITY API & SDK                      │
│                                                                  │
│  • Government ID Capture & Facial Biometrics processing          │
│  • Webhook Callback updates `Verifications` & `Users.Status`      │
└──────────────┬───────────────────────────────────────────────────┘
               │
               │ 2. Unlocks Feed (Buyers) or Triggers Subscription (Sellers)
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                   PAYSTACK PAYMENT GATEWAY                       │
│                                                                  │
│  • Business Subscription Purchase & Recurring Renewals           │
│  • Webhook Charge Success updates `Users.Is_Subscribed = True`    │
└──────────────┬───────────────────────────────────────────────────┘
               │
               │ 3. Unlocks Listing Creation & Publishing
               ▼
┌──────────────────────────────────────────────────────────────────┐
│                   ADALO BACKEND DATABASE                         │
│                                                                  │
│  • Verified Accounts, Active Subscriptions, Multi-Media Listings │
└──────────────────────────────────────────────────────────────────┘
🔄 Data Execution Lifecycle📝 Registration: User signs up and selects role (Buyer or Seller).🪪 Identity Verification: System dispatches a createVeriffSession API call and opens the Veriff webview. Upon approval, Users.Verification Status updates to Verified.🔓 Buyer Access: Verified Buyers gain immediate access to browse listings and initiate seller chats.💳 Seller Checkout: Verified Sellers are redirected to Paystack to complete business subscription payment. Upon charge success, Users.Is_Subscribed is set to True.🛍️ Publishing: Subscribed sellers gain access to create, update, and publish multi-media product listings.🛡️ 4. Security & Access Control🔐 Access Control MatrixUser Role / StatusBrowse ListingsContact Seller / ChatCreate & Publish ListingAccess Business Tools👤 Unverified User❌❌❌❌🪪 Verified Buyer✅✅❌❌💼 Verified Seller (Unsubscribed / Expired)✅✅❌❌⭐ Verified & Subscribed Seller✅✅✅✅🛡️ Component Visibility GuardsAdalo screen elements and action buttons enforce conditional rules evaluating logged-in user data:Marketplace View Access: Screen visibility requires Logged In User > Verification Status == "Verified".Create Listing Button: Button visibility requires Logged In User > User Type == "Seller" AND Verification Status == "Verified" AND Is_Subscribed == True AND Subscription End Date > Current Time.🔌 5. API Contract Specifications📥 5.1 Veriff Identity Verification API (createVeriffSession)Type: Custom Action / REST POSTEndpoint: [https://stationapi.veriff.com/v1/sessions](https://stationapi.veriff.com/v1/sessions)Access Level: Authenticated Registered UsersInput Payload:JSON{
  "verification": {
    "person": {
      "firstName": "{{ Logged In User > First Name }}",
      "lastName": "{{ Logged In User > Last Name }}"
    },
    "vendorData": "{{ Logged In User > Email }}"
  }
}
Response Payload (201 Created):JSON{
  "status": "success",
  "verification": {
    "id": "c1f2e8d3-4a5b-6c7d-8e9f-0a1b2c3d4e5f",
    "url": "https://alb.veriff.me/v/session_id"
  }
}
📤 5.2 Paystack Subscription Initialization API (initializePaystackPayment)Type: Custom Action / REST POSTEndpoint: [https://api.paystack.co/transaction/initialize](https://api.paystack.co/transaction/initialize)Access Level: Verified SellersInput Payload:JSON{
  "email": "{{ Logged In User > Email }}",
  "amount": "50000",
  "currency": "ZAR",
  "callback_url": "https://app.adalo.com/apps/marksold/payment-callback"
}
Execution Logic: Redirects user to data.authorization_url. Upon webhook notification of successful charge, Adalo sets Users.Is_Subscribed = True and extends Subscription End Date by 30 days.🚀 Setup & ConfigurationAdalo Database Setup:Replicate the collections structure (Users, Verifications, Listings, Payments, Chatrooms, Messages).Veriff Integration:Configure Custom Action in Adalo using your Veriff API credentials to generate verification sessions.Paystack Integration:Set up Paystack API Custom Actions for transaction initialization and map Webhook URLs in Paystack settings to update user subscription states automatically.
