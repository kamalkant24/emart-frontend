<template>
    <div class="privacy-wrapper">
      <div class="privacy-container">
        <h1 class="title">🛡️ Privacy Policy for Emart</h1>
        <p class="effective-date">Effective Date: January 1, 2025</p>
  
        <section v-for="(section, index) in policySections" :key="index" class="policy-section">
          <h2 class="section-title">📌 {{ section.title }}</h2>
          <ul v-if="section.points" class="point-list">
            <li v-for="(point, idx) in section.points" :key="idx" class="point">
              <span class="bullet-icon">✔</span> {{ point }}
            </li>
          </ul>
          <p v-else class="description">{{ section.description }}</p>
        </section>
  
        <section class="contact-section">
          <h2 class="section-title">📬 Contact Us</h2>
          <p>Email: <a href="mailto:inquiries@simplefixit.com">inquiries@simplefixit.com</a></p>
          <p>Address: SAS Nagar, Mohali, Punjab 160055</p>
        </section>
      </div>
    </div>
  </template>
  
  <script setup>
  const policySections = [
    {
      title: '1. Information We Collect',
      points: [
        'Personal Information: Name, email, address, phone number, payment info.',
        'Order Details: Purchased items, preferences, history.',
        'Technical Data: IP address, OS, browser, and device.',
        'Cookies & Usage Data: For better user experience and analytics.'
      ]
    },
    {
      title: '2. How We Use Your Information',
      points: [
        'To process orders and payments',
        'To deliver products and support',
        'To improve website performance',
        'To send updates/promotions',
        'To prevent fraud'
      ]
    },
    {
      title: '3. How We Share Your Information',
      points: [
        'With payment providers like Stripe/PayPal',
        'With shipping partners',
        'With tools like Google Analytics, Meta Pixel',
        'With legal authorities if required'
      ]
    },
    {
      title: '4. Data Security',
      points: [
        'SSL encryption',
        'Secure servers',
        'Regular backups',
        'Access control mechanisms'
      ]
    },
    {
      title: '5. Your Rights',
      points: [
        'Access your data',
        'Request correction or deletion',
        'Opt out of marketing',
        'Request data portability',
        'Contact: your-email@example.com'
      ]
    },
    {
      title: '6. Cookies & Tracking Technologies',
      points: [
        'Saving cart items',
        'Personalizing content',
        'Analyzing traffic',
        'Manage via browser settings'
      ]
    },
    {
      title: "7. Children's Privacy",
      description: 'We do not knowingly collect data from children under 13. If detected, it will be deleted immediately.'
    },
    {
      title: '8. Changes to This Policy',
      description: 'We may update this policy. The latest version will always be on this page.'
    }
  ];
  </script>
  
  <style scoped>
  /* Full Page Wrapper */
  .privacy-wrapper {
    min-height: 100vh;
    width: 100%;
    background: linear-gradient(to right, #f4f7fc, #eef2f7);
    display: flex;
    justify-content: center;
    align-items: flex-start;
    padding: 4rem 2rem;
    box-sizing: border-box;
  }
  
  /* Main Container */
  .privacy-container {
    /* max-width: 800px; */
    width: 100%;
    padding: 3rem;
    background: #ffffff;
    border-radius: 15px;
    box-shadow: 0 15px 30px rgba(0, 0, 0, 0.1);
    font-family: 'Inter', sans-serif;
    line-height: 1.8;
    color: #333;
    animation: fadeIn 0.8s ease-out;
  }
  
  /* Fade-in Animation */
  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  /* Titles */
  .title {
    font-size: 2.6rem;
    font-weight: 700;
    margin-bottom: 1rem;
    color: #1f2937;
  }
  
  .effective-date {
    font-size: 1.1rem;
    color: #6b7280;
    margin-bottom: 2rem;
    font-weight: 500;
  }
  
  /* Section Titles */
  .section-title {
    font-size: 1.6rem;
    color: #111827;
    font-weight: 600;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
  }
  
  .section-title .bullet-icon {
    margin-right: 0.5rem;
    font-size: 1.4rem;
  }
  
  /* Sections */
  .policy-section {
    margin-bottom: 2.5rem;
    padding: 1.5rem;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
  }
  
  .policy-section:hover {
    background-color: #f0f9ff;
  }
  
  /* List Styling */
  .point-list {
    padding-left: 0;
    list-style: none;
  }
  
  .point {
    margin-bottom: 0.7rem;
    display: flex;
    align-items: flex-start;
    font-size: 1.1rem;
    color: #374151;
  }
  
  .bullet-icon {
    color: #10b981;
    font-weight: bold;
    margin-right: 10px;
    font-size: 1.3rem;
    line-height: 1.3;
  }
  
  .description {
    font-size: 1.1rem;
    color: #4b5563;
    margin-bottom: 1rem;
  }
  
  /* Contact */
  .contact-section {
    margin-top: 3rem;
    padding-top: 1.5rem;
    border-top: 1px solid #e5e7eb;
  }
  
  a {
    color: #2563eb;
    font-weight: 500;
    text-decoration: none;
  }
  
  a:hover {
    text-decoration: underline;
  }
  
  /* Responsive */
  @media screen and (max-width: 768px) {
    .privacy-container {
      padding: 2rem;
    }
  
    .title {
      font-size: 2rem;
    }
  
    .section-title {
      font-size: 1.4rem;
    }
  
    .point {
      font-size: 1rem;
    }
  
    .description {
      font-size: 1rem;
    }
  }
  </style>
  