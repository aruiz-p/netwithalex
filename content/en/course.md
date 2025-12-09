---
# title: "Master Cisco SD-WAN UX 2.0 — Configuration & Policy Groups Explained"
# description: "Learn to confidently navigate Cisco SD-WAN UX 2.0 with configuration groups, policy groups, and topology management. Launch February 2026."
# date: 2024-12-02
url: /ux2-course/
draft: false
showToc: false
disableShare: false
hideMeta: false
ShowBreadCrumbs: false
ShowPostNavLinks: false
showReadingTime: false
cover:
    image: ""
    alt: ""
    relative: false
---

<style>
/* Override PaperMod Constraints - CRITICAL */
.main {
    max-width: none !important;
    width: 100% !important;
    padding: 0 !important;
    margin: 0 !important;
}

.post-content {
    max-width: none !important;
    width: 100% !important;
}

article {
    max-width: none !important;
}

/* Base Landing Page Styles */
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 0;
}

/* Full-Width Section Container */
.full-width-section {
    width: 100%;
    padding: 80px 5%;
}

.content-wrapper {
    max-width: 1200px;
    margin: 0 auto;
}

/* Hero Section - Full Width, Bold */
.hero-section {
    background: linear-gradient(130deg, #1B75BC 45%, #3A74C2 60%);
    color: white;
    text-align: center;
    padding: 150px 5% 150px;
    position: relative;
    overflow: hidden;
}

.hero-section::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: url('data:image/svg+xml,<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg"><defs><pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse"><path d="M 40 0 L 0 0 0 40" fill="none" stroke="rgba(255,255,255,0.05)" stroke-width="1"/></pattern></defs><rect width="100" height="100" fill="url(%23grid)"/></svg>');
    opacity: 0.4;
}

.hero-content {
    position: relative;
    z-index: 1;
    max-width: 900px;
    margin: 0 auto;
    color: white;
}

.hero-section h1 {
    font-size: 3rem;
    line-height: 1.2;
    margin-bottom: 25px;
    font-weight: 800;
    color: white;
}

.hero-section .subheadline {
    font-size: 1.4rem;
    margin-bottom: 40px;
    opacity: 0.95;
    font-weight: 400;
    line-height: 1.6;
}

.trust-badge {
    display: inline-block;
    background: rgba(255,255,255,0.25);
    backdrop-filter: blur(10px);
    padding: 12px 24px;
    border-radius: 30px;
    font-size: 1rem;
    margin-top: 20px;
    border: 1px solid rgba(255,255,255,0.3);
}

.cta-button {
    display: inline-block;
    background: white;
    color: #1B75BC;
    padding: 20px 45px;
    border-radius: 8px;
    font-size: 1.2rem;
    font-weight: 700;
    text-decoration: none;
    margin-top: 20px;
    transition: transform 0.3s, box-shadow 0.3s, background 0.3s;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.cta-button:hover {
    transform: translateY(-3px);
    box-shadow: 0 15px 40px rgba(0,0,0,0.3);
    background: #f0f0f0;
}

/* Alternating Sections */
.section-white {
    background: #ffffff;
}

.section-gray {
    background: #f8f9fa;
}

.section-light-blue {
    background: linear-gradient(135deg, #92BBF8 0%, #ACC8BA 100%);
    padding-bottom: 150px;
}

.section-dark {
    background: #1a1a1a;
    color: white;
}

.section h2 {
    font-size: 2.5rem;
    margin-bottom: 30px;
    color: #1B75BC;
    text-align: center;
}

.section-dark h2 {
    color: white;
}

/* Problem Section */
.pain-points {
    background: white;
    padding: 50px;
    border-radius: 12px;
    border: 3px solid #1B75BC;
    max-width: 900px;
    margin: 0 auto;
    box-shadow: 0 8px 30px rgba(27,117,188,0.15);
}

.pain-points ul {
    list-style: none;
    padding: 0;
    margin: 0;
}

.pain-points li {
    padding: 18px 0;
    font-size: 1.15rem;
    line-height: 1.7;
    border-bottom: 1px solid #f0f0f0;
}

.pain-points li:last-child {
    border-bottom: none;
}

.pain-points li:before {
    content: "❌ ";
    margin-right: 12px;
    font-size: 1.3rem;
}

/* Solution Box */
.solution-box {
    background: white;
    padding: 60px;
    border-radius: 12px;
    max-width: 900px;
    margin: 0 auto;
    box-shadow: 0 10px 40px rgba(0,0,0,0.08);
}

.solution-box h3 {
    color: #1B75BC;
    font-size: 2rem;
    margin-bottom: 25px;
    text-align: center;
}

.solution-box p {
    font-size: 1.2rem;
    line-height: 1.9;
    color: #333;
}

/* Course Value Container - Two Column Layout */
.course-value-container {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 60px;
    max-width: 1200px;
    margin: 50px auto;
    align-items: center;
}

/* Left Side - Course Preview */
.course-preview {
    position: relative;
}

.course-image-placeholder {
    background: white;
    border-radius: 30px;
    padding: 10px;
    padding-top: 0px;
    box-shadow: 0 15px 50px rgba(0,0,0,0.15);
}

.course-image-placeholder img {
    width: 100%;
    height: auto;
    border-radius: 8px;
    display: block;
}

.money-back-badge {
    background: linear-gradient(135deg, #1B75BC 0%, #3A74C2 100%);
    color: white;
    padding: 15px 25px;
    border-radius: 50px;
    text-align: center;
    font-weight: 700;
    font-size: 1.1rem;
    margin-top: 25px;
    box-shadow: 0 8px 20px rgba(27,117,188,0.3);
}

/* CTA Button in Course Section */
.course-cta-button {
    display: inline-block;
    background: #1B75BC;
    color: white;
    padding: 18px 40px;
    border-radius: 8px;
    font-size: 1.15rem;
    font-weight: 700;
    text-decoration: none;
    margin-top: 40px;
    transition: transform 0.3s, box-shadow 0.3s, background 0.3s;
    box-shadow: 0 8px 25px rgba(27,117,188,0.3);
    text-align: center;
}

.course-cta-button:hover {
    transform: translateY(-3px);
    box-shadow: 0 12px 35px rgba(27,117,188,0.4);
    background: #39527B;
    color: white;
    text-decoration: none;
}

/* Right Side - Value Props Grid */
.value-props-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 25px;
}

.value-prop-item {
    background: white;
    padding: 25px;
    border-radius: 12px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.08);
    transition: transform 0.3s, box-shadow 0.3s;
    display: flex;
    align-items: center;
    gap: 15px;
}

.value-prop-item:hover {
    transform: translateY(-3px);
    box-shadow: 0 8px 25px rgba(27,117,188,0.15);
}

.value-icon {
    font-size: 2.5rem;
    flex-shrink: 0;
}

.value-content {
    flex: 1;
}

.value-number {
    font-size: 1.2rem;
    font-weight: 700;
    color: #1B75BC;
    line-height: 1.3;
    margin-bottom: 3px;
}

.value-label {
    font-size: 0.95rem;
    color: #666;
    line-height: 1.4;
}

/* Screenshot Gallery */
.screenshot-gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
    gap: 30px;
    margin: 50px auto;
    max-width: 1200px;
}

.screenshot-gallery img {
    width: 100%;
    border-radius: 12px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.15);
    transition: transform 0.4s, box-shadow 0.4s;
}

.screenshot-gallery img:hover {
    transform: scale(1.03);
    box-shadow: 0 15px 45px rgba(0,0,0,0.25);
}

/* Module Cards */
.course-modules {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 30px;
    max-width: 1200px;
    margin: 40px auto;
}

.module-card {
    background: white;
    padding: 35px;
    border-radius: 12px;
    border: 2px solid #92BBF8;
    transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
}

.module-card:hover {
    transform: translateY(-5px);
    border-color: #1B75BC;
    box-shadow: 0 12px 30px rgba(27,117,188,0.2);
}

.module-card h4 {
    color: #1B75BC;
    font-size: 1.4rem;
    margin-bottom: 15px;
}

.module-card p {
    line-height: 1.7;
    color: #666;
}

/* Benefit Cards Grid - Three Column Layout */
.benefit-cards-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 30px;
    max-width: 1300px;
    margin: 0 auto;
}

.benefit-card {
    background: white;
    padding: 40px 30px;
    border-radius: 12px;
    text-align: center;
    box-shadow: 0 4px 15px rgba(0,0,0,0.08);
    transition: transform 0.3s, box-shadow 0.3s;
}

.benefit-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 12px 30px rgba(27,117,188,0.15);
}

.benefit-icon-circle {
    width: 100px;
    height: 100px;
    background: rgba(146,187,248,0.3);
    border: 0px solid #1B75BC;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    font-size: 3rem;
    margin-bottom: 25px;
    transition: transform 0.3s, background 0.3s;
}

.benefit-card:hover .benefit-icon-circle {
    transform: scale(1.05);
    background: rgba(146,187,248,0.5);
}

.benefit-title {
    font-size: 1.5rem;
    color: #405E7B;
    font-weight: 700;
    margin-bottom: 25px;
    line-height: 1.3;
}

.benefit-list {
    list-style: none;
    padding: 0;
    margin: 0;
    text-align: left;
}

.benefit-list li {
    padding: 12px 0 12px 30px;
    font-size: 1rem;
    line-height: 1.6;
    color: #333;
    position: relative;
}

.benefit-list li:before {
    content: "•";
    position: absolute;
    left: 10px;
    color: #1B75BC;
    font-weight: bold;
    font-size: 1.3rem;
}

/* About Section - Side by Side */
.about-section {
    display: grid;
    grid-template-columns: 300px 1fr;
    gap: 60px;
    align-items: center;
    max-width: 1100px;
    margin: 0 auto;
    padding: 150px 40px;
    background: white;
    border-radius: 12px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.08);
}

.about-image img {
    width: 300px;
    height: 300px;
    border-radius: 50%;
    object-fit: cover;
    border: 8px solid #92BBF8;
    box-shadow: 0 8px 25px rgba(27,117,188,0.2);
}

.about-content h3 {
    color: #405E7B;
    font-size: 2.2rem;
    margin-bottom: 20px;
}

.about-content p {
    font-size: 1.15rem;
    line-height: 1.9;
    color: #333;
    margin-bottom: 20px;
}

/* Pricing Box - Full Width Impact */
.pricing-section {
    background: linear-gradient(135deg, #1B75BC 0%, #39527B 100%);
    color: white;
    padding: 80px 5%;
    text-align: center;
    position: relative;
}

.pricing-box {
    max-width: 700px;
    margin: 0 auto;
}

.pricing-box h3 {
    font-size: 2.5rem;
    margin-bottom: 25px;
    color: white;
}

.price {
    font-size: 4rem;
    font-weight: 800;
    margin: 30px 0;
    line-height: 1;
}

.price-strike {
    text-decoration: line-through;
    opacity: 0.6;
    font-size: 2rem;
    display: block;
    margin-bottom: 10px;
}

.savings {
    background: rgba(255,255,255,0.25);
    backdrop-filter: blur(10px);
    display: inline-block;
    padding: 12px 28px;
    border-radius: 30px;
    margin-top: 15px;
    font-size: 1.1rem;
    border: 1px solid rgba(255,255,255,0.3);
}

/* Form Section - Wider Version */
.form-section {
    background: white;
    padding: 70px 50px;
    border-radius: 12px;
    box-shadow: 0 15px 50px rgba(0,0,0,0.12);
    max-width: 750px;
    margin: -80px auto 0;
    position: relative;
    z-index: 10;
}

.form-section h3 {
    text-align: center;
    color: #1B75BC;
    font-size: 2.2rem;
    margin-bottom: 20px;
}

.form-section > p {
    text-align: center;
    font-size: 1.1rem;
    margin-bottom: 40px;
    color: #666;
}

/* FAQ Section - Two Column Collapsible */
.faq-section {
    max-width: 1200px;
    margin: 0 auto;
}

.faq-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
}

.faq-item {
    margin: 0;
    background: white;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    overflow: hidden;
    transition: box-shadow 0.3s;
}

.faq-item:hover {
    box-shadow: 0 4px 15px rgba(0,0,0,0.12);
}

.faq-question {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 25px 30px;
    cursor: pointer;
    background: white;
    border: none;
    width: 100%;
    text-align: left;
    transition: background 0.3s;
    gap: 15px;
}

.faq-question:hover {
    background: #f8f9fa;
}

.faq-question-text {
    display: flex;
    align-items: center;
    gap: 12px;
    flex: 1;
}

.faq-emoji {
    font-size: 1.5rem;
    flex-shrink: 0;
}

.faq-question h4 {
    color: #405E7B;
    font-size: 1.1rem;
    margin: 0;
    font-weight: 600;
    line-height: 1.4;
}

.faq-toggle {
    font-size: 1.5rem;
    color: #1B75BC;
    transition: transform 0.3s;
    flex-shrink: 0;
}

.faq-item.active .faq-toggle {
    transform: rotate(180deg);
}

.faq-answer {
    max-height: 0;
    overflow: hidden;
    transition: max-height 0.3s ease-out, padding 0.3s ease-out;
    padding: 0 30px;
}

.faq-item.active .faq-answer {
    max-height: 500px;
    padding: 0 30px 25px 30px;
}

.faq-answer p {
    font-size: 1rem;
    line-height: 1.7;
    color: #666;
    margin: 0;
    padding-left: 42px;
}

/* Final CTA */
.final-cta {
    background: linear-gradient(135deg, #39527B 0%, #234062 100%);
    color: white;
    text-align: center;
    padding: 80px 5%;
}

.final-cta h2 {
    color: white;
    font-size: 2.8rem;
    margin-bottom: 25px;
}

.final-cta p {
    font-size: 1.3rem;
    margin-bottom: 35px;
    opacity: 0.95;
}

/* Responsive Design */
@media (max-width: 1024px) {
    .about-section {
        grid-template-columns: 1fr;
        text-align: center;
    }
    
    .about-image img {
        margin: 0 auto;
    }
    
    .course-value-container {
        grid-template-columns: 1fr;
        gap: 40px;
    }
    
    .value-props-grid {
        grid-template-columns: 1fr;
    }
    
    .benefit-cards-grid {
        grid-template-columns: 1fr;
    }
}

@media (max-width: 768px) {
    .hero-section h1 {
        font-size: 2rem;
    }
    
    .hero-section .subheadline {
        font-size: 1.1rem;
    }
    
    .section h2 {
        font-size: 2rem;
    }
    
    .full-width-section {
        padding: 50px 5%;
    }
    
    .pain-points, .solution-box {
        padding: 30px;
    }
    
    .form-section {
        padding: 40px 25px;
        margin-top: -60px;
        max-width: 95%;
    }
    
    .screenshot-gallery {
        grid-template-columns: 1fr;
    }
    
    .course-modules {
        grid-template-columns: 1fr;
    }
    
    .price {
        font-size: 3rem;
    }
    
    .faq-grid {
        grid-template-columns: 1fr;
    }
    
    .faq-question {
        padding: 20px;
    }
    
    .faq-item.active .faq-answer {
        padding: 0 20px 20px 20px;
    }
    
    .faq-answer p {
        padding-left: 35px;
    }
}
</style>

<!-- Hero Section - Full Width -->
<div class="hero-section">
    <div class="hero-content">
        <h1>Online Course: Cisco SD-WAN UX 2.0</h1>
        <p class="subheadline">Master Configuration and Policy Groups without the confusion or trial-and-error.</p>
        <a href="#waitlist" class="cta-button">Join the Waitlist — Get Early Access</a><br><br>
        <div class="savings">💰 Save $30 — Waitlist Members Only</div>
        <p style="margin-top: 35px; font-size: 1.15rem;">
            Regular price: $97 after launch<br>
            Early bird ends when the course goes live in February 2026
        </p>
    </div>
</div>

<!-- Form Section - Overlapping Design -->
<div class="full-width-section section-gray" style="padding-top: 200px;">
    <div class="form-section" id="waitlist">
        <h3>Join the Waitlist</h3>
        <p>Get notified when the course launches in February 2026, lock in the $67 early bird price, and receive exclusive bonus resources.</p>
        <!-- Kit Form Embed -->
        <script async data-uid="bb5c706e22" src="https://netwithalex.kit.com/bb5c706e22/index.js"></script>
    </div>
</div>



<!-- About Alex - White Background -->
<div class="full-width-section section-white">
    <div class="about-section">
        <div class="about-image">
            <img src="/wp-content/uploads/course-ux/alex-pic.png" alt="Alex - Network Engineer and CCIE" />
        </div>
        <div class="about-content">
            <h3>Learn From Real, Everyday Experience</h3>
            <p>
                Hi, I'm Alex. I'm a CCIE Enterprise certified network engineer with 8+ years in the field, sharing hands-on SD-WAN knowledge with hundreds of engineers worldwide through NetWithAlex.
            </p>
            <p>
                I've helped teams navigate complex Cisco SD-WAN deployments, and I know exactly where engineers get stuck with UX 2.0. This course gives you the clarity and confidence to master the new interface without the frustration.
            </p>
        </div>
    </div>
</div>

<!-- Who This Is For - Light Blue Background -->
<div class="full-width-section section-light-blue">
    <div class="content-wrapper">
        <h2 style="color: #405E7B; text-align:center;">This course is for you if...</h2><br><br>
        <div class="benefit-cards-grid">
            <div class="benefit-card">
                <div class="benefit-icon-circle">♻️</div>
                <h3 class="benefit-title">Migrating from Legacy</h3>
                <p style="padding-left:10px; text-align: left;">You're running the template-based system and need to understand the UX 2.0 constructs and the advantages it provides.</p>
            </div>
            <div class="benefit-card">
                <div class="benefit-icon-circle">🆕</div>
                <h3 class="benefit-title">Starting Fresh</h3>
                <p style="padding-left:10px; text-align: left;">You're new to Cisco SD-WAN or want to skip legacy methods and go straight to the modern approach that will let you get the most out of the solution.</p>
            </div>
            <div class="benefit-card">
                <div class="benefit-icon-circle">📚</div>
                <h3 class="benefit-title">Exam Prep</h3>
                <p style="padding-left:10px; text-align: left;">You're studying for certifications and need practical understanding beyond theory that will not only make knowledge retention easier, but will prepare you for real life scenarios.</p>
            </div>
        </div>
    </div>
</div>

<!-- What's Included - Gray Background -->
<div class="full-width-section section-gray">
    <div class="content-wrapper">
        <h2 style="text-align:center; color: #1B75BC">What's Inside the Course</h2>
        <div class="course-value-container">
            <!-- Left Side: Course Preview Image -->
            <div class="course-preview">
                <div class="course-image-placeholder">
                    <img src="/wp-content/uploads/course-ux/course-comp.png" alt="Course Preview - Multiple devices showing course content" />
                </div>
                <div class="money-back-badge">
                    💙 30-Day Money-Back Guarantee
                </div>
            </div>
            <!-- Right Side: Value Props Grid -->
            <div class="value-props-grid">
                <div class="value-prop-item">
                    <div class="value-icon">⏰</div>
                    <div class="value-content">
                        <div class="value-number">3+ hours</div>
                        <div class="value-label">of focused video content</div>
                    </div>
                </div>
                <div class="value-prop-item">
                    <div class="value-icon">🎯</div>
                    <div class="value-content">
                        <div class="value-number">15+ lessons</div>
                        <div class="value-label">step-by-step guidance</div>
                    </div>
                </div>
                <div class="value-prop-item">
                    <div class="value-icon">🧠</div>
                    <div class="value-content">
                        <div class="value-number">UX 2.0 Constructs</div>
                        <div class="value-label">complete deep-dive</div>
                    </div>
                </div>
                <div class="value-prop-item">
                    <div class="value-icon">🔧</div>
                    <div class="value-content">
                        <div class="value-number">Lab files included</div>
                        <div class="value-label">running configs & topologies</div>
                    </div>
                </div>
                <div class="value-prop-item">
                    <div class="value-icon">📄</div>
                    <div class="value-content">
                        <div class="value-number">PDF guides</div>
                        <div class="value-label">quick reference materials</div>
                    </div>
                </div>
                <div class="value-prop-item">
                    <div class="value-icon">✉️</div>
                    <div class="value-content">
                        <div class="value-number">Email support</div>
                        <div class="value-label">when you need help</div>
                    </div>
                </div>
            </div>
        </div>
        <!-- CTA Button Below Course Content -->
        <div style="text-align: center; margin-top: 50px;">
            <a href="#waitlist" class="course-cta-button">Join the Waitlist Now</a>
        </div>
    </div>
</div>

<!-- FAQ - Light Blue Background -->
<div class="full-width-section section-light-blue" style="padding-top: 40px;">
    <div class="content-wrapper">
        <h2 style="color: #405E7B;">Frequently Asked Questions 💡</h2>
        <br>
        <div class="faq-section">
            <div class="faq-grid">
                <!-- FAQ Item 1 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">📅</span>
                            <h4>When does the course launch?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>The course will be available on <strong>February 1, 2026</strong>. Waitlist members will be notified first and get exclusive early bird pricing ($67 vs $97 regular price).</p>
                    </div>
                </div>
                <!-- FAQ Item 2 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">🎓</span>
                            <h4>Is this course good for beginners?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>Yes! While the course focuses on configuration management in UX 2.0, it's designed to be accessible whether you're new to SD-WAN or migrating from legacy templates. The concepts are explained from the ground up with practical examples.</p>
                    </div>
                </div>
                <!-- FAQ Item 3 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">💻</span>
                            <h4>What do I need to follow along?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>The course is easiest to follow if you have access to <strong>Cisco Modeling Labs (CML)</strong>, but you don't need it. I'll provide running configs so you can build the topology in EVE-NG, GNS3, or any virtualization platform you prefer. You can even just watch and learn without a lab.</p>
                    </div>
                </div>
                <!-- FAQ Item 4 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">✉️</span>
                            <h4>What if I have questions during the course?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>You'll have <strong>email support</strong> included. If you get stuck or need clarification on any topic, just reach out and I'll help you work through it.</p>
                    </div>
                </div>
                <!-- FAQ Item 5 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">⏰</span>
                            <h4>How long will I have access to the course?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>You get <strong>lifetime access</strong> to all course materials, including any future updates. Buy once, learn at your own pace, reference forever.</p>
                    </div>
                </div>
                <!-- FAQ Item 6 -->
                <div class="faq-item active">
                    <button class="faq-question" onclick="toggleFAQ(this)">
                        <div class="faq-question-text">
                            <span class="faq-emoji">🔄</span>
                            <h4>What if UX 2.0 changes after I buy the course?</h4>
                        </div>
                        <span class="faq-toggle">−</span>
                    </button>
                    <div class="faq-answer">
                        <p>Cisco SD-WAN UX 2.0 is stable and will be the standard interface going forward. If there are significant updates, I'll add new content to keep the course current — at no additional cost to you.</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
function toggleFAQ(button) {
    const faqItem = button.parentElement;
    const isActive = faqItem.classList.contains('active');
    const toggle = button.querySelector('.faq-toggle');
    
    // Toggle just this item (don't affect others)
    if (isActive) {
        faqItem.classList.remove('active');
        toggle.textContent = '+';
    } else {
        faqItem.classList.add('active');
        toggle.textContent = '−';
    }
}
</script>

<!-- Final CTA - Dark Blue -->
<div class="final-cta">
    <div class="content-wrapper">
        <h2>Ready to Master UX 2.0?</h2>
        <p>
            Join the waitlist now and be the first to know when early bird pricing opens in February 2026.
        </p>
        <a href="#waitlist" class="cta-button">Get Early Access — Save $30</a>
    </div>
    <div class="trust-badge">✨ Launching February 2026 | Early Bird: $67 (Save $30)</div>
    </div>
</div>

<!-- Pricing - Full Width Blue
<div class="pricing-section">
    <div class="pricing-box">
        <h3>Special Early Bird Pricing</h3>
        <p style="font-size: 1.3rem; opacity: 0.95;">Join the waitlist and lock in exclusive early access pricing:</p>
        <div class="price">
            <span class="price-strike">$97</span>
            $67
        </div>
        <div class -->