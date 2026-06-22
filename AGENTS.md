DESCRIPTION: PJP QRIS is a Vue 3 web app used to check the PJP (Penyelenggara Jasa Pembayaran) using the NNS code or id.

INSTRUCTIONS:
- Build this web app using Vue 3, the node.js library commonly used to build reactive web apps.
- The web app should be named: PJP QRIS Check.
- The interface should be very straightforward following the INTERFACE_INSTRUCTIONS.
- The user flow is defined in the FLOW section.

INTERFACE_INSTRUCTIONS:
- The UI is build using the latest version of Tailwindcss
- The UI should be simple, only 3 main sections:
  - First, the section that contains the how-to know where to find the NNS code within a QRIS: tell the user to look for a text on the QRIS that says "Dicetak oleh: {their_merchant_NNS}".
  - Second, the section that contains the title of the web app and below it the input to type the NNS code.
  - Third, the section that contains company names and their NNS codes of BRI, Mandiri, BCA, BNI, GoPay, OVO, DANA, ShopeePay, LinkAja in their corresponding order. Also, the footer and the information of the updated data of the data.
- The UI should be dark and have vibrant colors (mainly purple or violer color pallete) while maintaining an easy and simple navigation without navbar.

FLOW:
1. User follows the instruction to get their NNS code within their QRIS.
2. User inputs the NNS into the input and get the corresponding information about the NNS, nama_penyelenggara, and nama_produk from the pjp_qris.json file.
