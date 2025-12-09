---
title: "Master Cisco SD-WAN UX 2.0 — Configuration & Policy Groups Explained"
description: "Learn to confidently navigate Cisco SD-WAN UX 2.0 with configuration groups, policy groups, and topology management. Launch February 2026."
date: 2024-12-02
draft: false
name: UX2-Course
url: /course-ux2/
lang: en
showToc: false
disableShare: false
hideMeta: false
ShowBreadCrumbs: false
ShowPostNavLinks: false
cover:
    image: ""
    alt: ""
    relative: false
---

<style>
/* Landing Page Styles */
.landing-container {
    max-width: 900px;
    margin: 0 auto;
    padding: 0;
}

.hero-section {
    text-align: center;
    padding: 60px 20px 40px;
    background: linear-gradient(135deg, #0078D4 0%, #005A9E 100%);
    color: white;
    border-radius: 8px;
    margin-bottom: 50px;
}

.hero-section h1 {
    font-size: 2.5rem;
    line-height: 1.2;
    margin-bottom: 20px;
    font-weight: 700;
}

.hero-section .subheadline {
    font-size: 1.3rem;
    margin-bottom: 30px;
    opacity: 0.95;
    font-weight: 400;
}

.trust-badge {
    display: inline-block;
    background: rgba(255,255,255,0.2);
    padding: 10px 20px;
    border-radius: 30px;
    font-size: 0.95rem;
    margin-top: 20px;
}

.cta-button {
    display: inline-block;
    background: white;
    color: #0078D4;
    padding: 18px 40px;
    border-radius: 6px;
    font-size: 1.1rem;
    font-weight: 600;
    text-decoration: none;
    margin-top: 20px;
    transition: transform 0.2s, box-shadow 0.2s;
}

.cta-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(0,0,0,0.2);
}

.section {
    margin: 60px 0;
    padding: 0 20px;
}

.section h2 {
    font-size: 2rem;
    margin-bottom: 25px;
    color: #0078D4;
}

.pain-points {
    background: #f8f9fa;
    padding: 40px;
    border-radius: 8px;
    border-left: 5px solid #0078D4;
}

.pain-points ul {
    list-style: none;
    padding: 0;
}

.pain-points li {
    padding: 12px 0;
    font-size: 1.1rem;
    line-height: 1.6;
}

.pain-points li:before {
    content: "❌ ";
    margin-right: 10px;
}

.solution-box {
    background: linear-gradient(135deg, #E8F4FD 0%, #D1E7F7 100%);
    padding: 40px;
    border-radius: 8px;
    margin: 30px 0;
}

.solution-box h3 {
    color: #0078D4;
    font-size: 1.5rem;
    margin-bottom: 20px;
}

.benefits-list li {
    padding: 10px 0;
    font-size: 1.05rem;
    line-height: 1.7;
}

.benefits-list li:before {
    content: "✓ ";
    color: #0078D4;
    font-weight: bold;
    margin-right: 10px;
}

.course-modules {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin: 30px 0;
}

.module-card {
    background: white;
    padding: 25px;
    border-radius: 8px;
    border: 2px solid #E8F4FD;
    transition: border-color 0.3s, box-shadow 0.3s;
}

.module-card:hover {
    border-color: #0078D4;
    box-shadow: 0 4px 12px rgba(0,120,212,0.1);
}

.module-card h4 {
    color: #0078D4;
    font-size: 1.2rem;
    margin-bottom: 12px;
}

.about-section {
    display: flex;
    align-items: center;
    gap: 40px;
    background: #f8f9fa;
    padding: 40px;
    border-radius: 8px;
    margin: 50px 0;
}

.about-image {
    flex-shrink: 0;
}

.about-image img {
    width: 200px;
    height: 200px;
    border-radius: 50%;
    object-fit: cover;
    border: 5px solid white;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.about-content {
    flex: 1;
}

.about-content h3 {
    color: #0078D4;
    font-size: 1.8rem;
    margin-bottom: 15px;
}

.pricing-box {
    background: linear-gradient(135deg, #0078D4 0%, #005A9E 100%);
    color: white;
    padding: 40px;
    border-radius: 8px;
    text-align: center;
    margin: 40px 0;
}

.pricing-box h3 {
    font-size: 2rem;
    margin-bottom: 20px;
}

.price {
    font-size: 3rem;
    font-weight: 700;
    margin: 20px 0;
}

.price-strike {
    text-decoration: line-through;
    opacity: 0.7;
    font-size: 1.5rem;
    display: block;
}

.savings {
    background: rgba(255,255,255,0.2);
    display: inline-block;
    padding: 10px 20px;
    border-radius: 30px;
    margin-top: 10px;
}

.form-section {
    background: white;
    padding: 50px;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.1);
    margin: 40px 0;
}

.form-section h3 {
    text-align: center;
    color: #0078D4;
    font-size: 2rem;
    margin-bottom: 30px;
}

.faq-section {
    margin: 60px 0;
}

.faq-item {
    margin: 25px 0;
    padding: 25px;
    background: #f8f9fa;
    border-radius: 8px;
}

.faq-item h4 {
    color: #0078D4;
    font-size: 1.3rem;
    margin-bottom: 12px;
}

.faq-item p {
    font-size: 1.05rem;
    line-height: 1.7;
}

.screenshot-gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 20px;
    margin: 30px 0;
}

.screenshot-gallery img {
    width: 100%;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    transition: transform 0.3s;
}

.screenshot-gallery img:hover {
    transform: scale(1.02);
}

@media (max-width: 768px) {
    .hero-section h1 {
        font-size: 1.8rem;
    }
    
    .hero-section .subheadline {
        font-size: 1.1rem;
    }
    
    .about-section {
        flex-direction: column;
        text-align: center;
    }
    
    .course-modules {
        grid-template-columns: 1fr;
    }
    
    .form-section {
        padding: 30px 20px;
    }
}
</style>

<div class="landing-container">

<!-- Hero Section -->
<div class="hero-section">
    <h1>Stop Struggling with Cisco SD-WAN UX 2.0</h1>
    <p class="subheadline">Master Configuration Groups, Policy Groups, and the new topology management in just 3 hours — without the confusion or trial-and-error.</p>
    <a href="#waitlist" class="cta-button">Join the Waitlist — Get Early Access</a>
    <div class="trust-badge"> Launching February 2026 | Early Bird: $67 (Save $30)</div>
</div>

<!-- Problem Section -->
<div class="section">
    <h2>Does This Sound Familiar?</h2>
    <div class="pain-points">
        <ul>
            <li>You've heard about UX 2.0 but the new interface feels completely different from what you know</li>
            <li>Configuration Groups and Policy Groups sound great in theory, but you're not sure how to actually implement them</li>
            <li>You're worried about migrating from legacy templates without breaking production</li>
            <li>Cisco's documentation is overwhelming and doesn't show real-world examples</li>
            <li>You need to get up to speed fast but don't have time for trial-and-error in the lab</li>
        </ul>
    </div>
</div>

<!-- Solution Section -->
<div class="section">
    <h2>Here's the Solution</h2>
    <div class="solution-box">
        <h3>A Practical, Hands-On Course Built for Network Engineers</h3>
        <p style="font-size: 1.1rem; line-height: 1.8;">
            <strong>Mastering Cisco SD-WAN UX 2.0</strong> takes you from confused to confident in just 3 hours. You'll learn the core configuration management concepts through real examples, not just theory.
        </p>
        <p style="font-size: 1.1rem; line-height: 1.8; margin-top: 20px;">
            This isn't another boring overview. You'll see exactly how Configuration Groups and Policy Groups work, follow along with actual deployments, and walk away ready to implement UX 2.0 in your own environment.
        </p>
    </div>
</div>

<!-- What's Included -->
<div class="section">
    <h2>What's Inside the Course</h2>
    <ul class="benefits-list">
        <li><strong>3 hours of focused video content</strong> — No fluff, just practical guidance you can apply immediately</li>
        <li><strong>Configuration Groups deep-dive</strong> — Understand feature templates, site configurations, and device associations</li>
        <li><strong>Policy Groups explained</strong> — Learn how to structure and apply policies in the new framework</li>
        <li><strong>Topology management</strong> — Navigate the new interface and visualize your SD-WAN fabric</li>
        <li><strong>Real-world examples</strong> — Follow along as we build configurations from scratch</li>
        <li><strong>Lab files included</strong> — Running configs you can use in CML, EVE-NG, or your platform of choice</li>
        <li><strong>PDF reference guides</strong> — Quick-reference materials for future deployments</li>
        <li><strong>Email support</strong> — Get your questions answered as you work through the content</li>
    </ul>
</div>

<!-- Screenshots -->
<div class="section">
    <h2>A Sneak Peek at What You'll Learn</h2>
    <div class="screenshot-gallery">
        <img src="/images/ux2-topology.png" alt="SD-WAN UX 2.0 Topology Diagram" />
        <img src="/images/ux2-config-groups.png" alt="Configuration Groups Interface" />
        <img src="/images/ux2-concept.png" alt="Configuration Groups Concept" />
    </div>
    <p style="text-align: center; margin-top: 20px; color: #666;">
        <em>Real screenshots from the course showing topology design, configuration group creation, and core concepts.</em>
    </p>
</div>

<!-- Who This Is For -->
<div class="section">
    <h2>Who This Course Is For</h2>
    <div class="course-modules">
        <div class="module-card">
            <h4>🔄 Migrating from Legacy</h4>
            <p>You're running the old template-based system and need to understand how UX 2.0 changes your workflow.</p>
        </div>
        <div class="module-card">
            <h4>🆕 Starting Fresh</h4>
            <p>You're new to Cisco SD-WAN or want to skip legacy methods and go straight to the modern approach.</p>
        </div>
        <div class="module-card">
            <h4>🎓 Exam Prep</h4>
            <p>You're studying for certifications and need practical understanding beyond theory.</p>
        </div>
        <div class="module-card">
            <h4>💼 Enterprise Deployments</h4>
            <p>You're responsible for SD-WAN in production and need confidence before making changes.</p>
        </div>
    </div>
</div>

<!-- About Alex -->
<div class="about-section">
    <div class="about-image">
        <img src="/images/alex-headshot.jpg" alt="Alex - Network Engineer and CCIE" />
    </div>
    <div class="about-content">
        <h3>Learn from a CCIE-Certified Engineer</h3>
        <p style="font-size: 1.1rem; line-height: 1.8; margin-bottom: 20px;">
            Hi, I'm Alex. I'm a CCIE Enterprise certified network engineer with 8+ years in the field, sharing hands-on SD-WAN knowledge with thousands of engineers worldwide through NetWithAlex.
        </p>
        <p style="font-size: 1.1rem; line-height: 1.8;">
            I've helped teams navigate complex Cisco SD-WAN deployments, and I know exactly where engineers get stuck with UX 2.0. This course gives you the clarity and confidence to master the new interface without the frustration.
        </p>
    </div>
</div>

<!-- Pricing -->
<div class="pricing-box">
    <h3>Special Early Bird Pricing</h3>
    <p style="font-size: 1.2rem; opacity: 0.95;">Join the waitlist and lock in exclusive early access pricing:</p>
    <div class="price">
        <span class="price-strike">$97</span>
        $67
    </div>
    <div class="savings">💰 Save $30 — Waitlist Members Only</div>
    <p style="margin-top: 30px; font-size: 1.1rem;">
        Regular price: $97 after launch<br>
        Early bird ends when the course goes live in February 2026
    </p>
</div>

<!-- CTA + Form -->
<div class="form-section" id="waitlist">
    <h3>Join the Waitlist — Be First to Get Early Access</h3>
    <p style="text-align: center; font-size: 1.1rem; margin-bottom: 30px; color: #666;">
        Get notified when the course launches in February 2026, lock in the $67 early bird price, and receive exclusive bonus resources.
    </p>
    <!-- Kit Form Embed -->
    <script async data-uid="242d71c937" src="https://netwithalex.kit.com/242d71c937/index.js"></script>
    <p style="text-align: center; margin-top: 20px; color: #999; font-size: 0.95rem;">
        🔒 Your email is safe. No spam, just course updates and valuable SD-WAN content.
    </p>
</div>

<!-- FAQ -->
<div class="section faq-section">
    <h2>Frequently Asked Questions</h2>
    <div class="faq-item">
        <h4>When does the course launch?</h4>
        <p>The course will be available on <strong>February 1, 2026</strong>. Waitlist members will be notified first and get exclusive early bird pricing ($67 vs $97 regular price).</p>
    </div>
    <div class="faq-item">
        <h4>Is this course good for beginners?</h4>
        <p>Yes! While the course focuses on configuration management in UX 2.0, it's designed to be accessible whether you're new to SD-WAN or migrating from legacy templates. The concepts are explained from the ground up with practical examples.</p>
    </div>
    <div class="faq-item">
        <h4>What do I need to follow along?</h4>
        <p>The course is easiest to follow if you have access to <strong>Cisco Modeling Labs (CML)</strong>, but you don't need it. I'll provide running configs so you can build the topology in EVE-NG, GNS3, or any virtualization platform you prefer. You can even just watch and learn without a lab.</p>
    </div>
    <div class="faq-item">
        <h4>What if I have questions during the course?</h4>
        <p>You'll have <strong>email support</strong> included. If you get stuck or need clarification on any topic, just reach out and I'll help you work through it.</p>
    </div>
    <div class="faq-item">
        <h4>How long will I have access to the course?</h4>
        <p>You get <strong>lifetime access</strong> to all course materials, including any future updates. Buy once, learn at your own pace, reference forever.</p>
    </div>
    <div class="faq-item">
        <h4>What if UX 2.0 changes after I buy the course?</h4>
        <p>Cisco SD-WAN UX 2.0 is stable and will be the standard interface going forward. If there are significant updates, I'll add new content to keep the course current — at no additional cost to you.</p>
    </div>
</div>

<!-- Final CTA -->
<div class="hero-section" style="margin-top: 60px;">
    <h2 style="color: white; margin-bottom: 20px;">Ready to Master UX 2.0?</h2>
    <p style="font-size: 1.2rem; margin-bottom: 30px; opacity: 0.95;">
        Join the waitlist now and be the first to know when early bird pricing opens in February 2026.
    </p>
    <a href="#waitlist" class="cta-button">Get Early Access — Save $30</a>
</div>

</div>