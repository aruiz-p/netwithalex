---
title: "Resources"
date: 2024-12-02
url: /resources/
draft: false
showToc: false
tocOpen: true
disableShare: false
hideMeta: false
ShowBreadCrumbs: true
ShowPostNavLinks: false
---

<style>
/* Resources Page Styles */
.resources-container {
    max-width: 1100px;
    margin: 0 auto;
}

.resources-intro {
    text-align: center;
    padding: 40px 20px;
    margin-bottom: 50px;
    background: linear-gradient(135deg, #E8F4FD 0%, #D1E7F7 100%);
    border-radius: 8px;
}

.resources-intro h1 {
    color: #0078D4;
    font-size: 2.5rem;
    margin-bottom: 15px;
}

.resources-intro p {
    font-size: 1.2rem;
    color: #555;
    max-width: 700px;
    margin: 0 auto;
    line-height: 1.7;
}

.resource-section {
    margin: 60px 0;
}

.resource-section h2 {
    color: #0078D4;
    font-size: 2rem;
    margin-bottom: 30px;
    padding-bottom: 10px;
    border-bottom: 3px solid #0078D4;
}

.resource-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 30px;
    margin-top: 30px;
}

.resource-card {
    background: white;
    border: 2px solid #E8F4FD;
    border-radius: 8px;
    padding: 30px;
    transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
    position: relative;
}

.resource-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 20px rgba(0,120,212,0.15);
    border-color: #0078D4;
}

.resource-badge {
    position: absolute;
    top: 15px;
    right: 15px;
    background: #0078D4;
    color: white;
    padding: 6px 14px;
    border-radius: 20px;
    font-size: 0.85rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.resource-badge.free {
    background: #28a745;
}

.resource-badge.coming-soon {
    background: #ffc107;
    color: #333;
}

.resource-badge.price {
    background: #0078D4;
    font-size: 1rem;
}

.resource-icon {
    font-size: 3rem;
    margin-bottom: 20px;
    display: block;
}

.resource-card h3 {
    color: #0078D4;
    font-size: 1.5rem;
    margin-bottom: 15px;
    line-height: 1.3;
}

.resource-card p {
    color: #666;
    font-size: 1.05rem;
    line-height: 1.7;
    margin-bottom: 20px;
}

.resource-meta {
    display: flex;
    gap: 15px;
    margin: 15px 0;
    font-size: 0.9rem;
    color: #888;
}

.resource-meta span {
    display: flex;
    align-items: center;
    gap: 5px;
}

.resource-cta {
    display: inline-block;
    background: #0078D4;
    color: white;
    padding: 12px 24px;
    border-radius: 6px;
    text-decoration: none;
    font-weight: 600;
    transition: background 0.3s;
    margin-top: 10px;
}

.resource-cta:hover {
    background: #005A9E;
    text-decoration: none;
    color: white;
}

.resource-cta.secondary {
    background: transparent;
    color: #0078D4;
    border: 2px solid #0078D4;
}

.resource-cta.secondary:hover {
    background: #0078D4;
    color: white;
}

.pdf-list {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 25px;
    margin-top: 30px;
}

.pdf-item {
    background: white;
    border: 2px solid #E8F4FD;
    padding: 25px;
    border-radius: 8px;
    transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
    position: relative;
}

.pdf-item:hover {
    transform: translateY(-3px);
    box-shadow: 0 6px 16px rgba(0,120,212,0.12);
    border-color: #0078D4;
}

.pdf-badge {
    position: absolute;
    top: 15px;
    right: 15px;
    background: #0078D4;
    color: white;
    padding: 6px 14px;
    border-radius: 20px;
    font-size: 1rem;
    font-weight: 700;
}

.pdf-badge.free {
    background: #28a745;
    text-transform: uppercase;
    font-size: 0.85rem;
}

.pdf-item h4 {
    color: #0078D4;
    font-size: 1.3rem;
    margin-bottom: 12px;
    padding-right: 60px; /* Space for badge */
}

.pdf-item p {
    font-size: 1rem;
    color: #666;
    margin-bottom: 20px;
    line-height: 1.6;
}

.pdf-meta {
    display: flex;
    gap: 15px;
    margin-bottom: 15px;
    font-size: 0.9rem;
    color: #888;
}

.pdf-meta span {
    display: flex;
    align-items: center;
    gap: 5px;
}

.pdf-download {
    display: inline-block;
    background: #0078D4;
    color: white;
    padding: 10px 20px;
    border-radius: 6px;
    text-decoration: none;
    font-weight: 600;
    transition: background 0.3s;
}

.pdf-download:hover {
    background: #005A9E;
    text-decoration: none;
    color: white;
}

.coming-soon-notice {
    background: #FFF9E6;
    border-left: 4px solid #ffc107;
    padding: 20px;
    margin: 20px 0;
    border-radius: 6px;
}

.coming-soon-notice p {
    margin: 0;
    color: #856404;
}

@media (max-width: 768px) {
    .resource-grid {
        grid-template-columns: 1fr;
    }
    
    .pdf-list {
        grid-template-columns: 1fr;
    }
    
    .resources-intro h1 {
        font-size: 2rem;
    }
}
</style>

<div class="resources-container">

<div class="resources-intro">
    <p>📚 Practical guides, courses, and tools to help you master Cisco SD-WAN and enterprise networking. All content is built from real-world experience.</p>
</div>

<!-- Courses Section -->
<div class="resource-section">
    <h2>🎓 Courses</h2>
    <div class="resource-grid">
        <!-- Course 1: UX 2.0 -->
        <div class="resource-card">
            <span class="resource-badge coming-soon">Coming Feb 2026</span>
            <span class="resource-icon">🚀</span>
            <h3>Mastering Cisco SD-WAN UX 2.0</h3>
            <p>Learn Configuration Groups, Policy Groups, and the new topology management. Go from confused to confident in 3 hours.</p>
            <div class="resource-meta">
                <span>⏱️ 3 hours</span>
                <span>📹 Video + PDFs</span>
            </div>
            <p style="font-size: 0.95rem; color: #888; margin-top: 15px;">
                <strong>What's Included:</strong><br>
                • Configuration Groups deep-dive<br>
                • Policy Groups explained<br>
                • Lab files & running configs<br>
                • Email support included
            </p>
            <div class="coming-soon-notice">
                <p><strong>⚡ Early Bird Pricing:</strong> Join the waitlist and lock in $67 (save $30 off regular $97 price)</p>
            </div>
            <a href="/ux2-course" class="resource-cta">Join the Waitlist</a>
        </div>
        <!-- Placeholder for Future Course 2 -->
        <div class="resource-card" style="opacity: 0.7; border-style: dashed;">
            <span class="resource-badge coming-soon">Coming Soon</span>
            <span class="resource-icon">🔧</span>
            <h3>Advanced SD-WAN Troubleshooting</h3>
            <p>Master the art of diagnosing and fixing SD-WAN issues in production. Real scenarios, proven methodologies.</p>
            <div class="resource-meta">
                <span>⏱️ TBD</span>
                <span>📹 Video + Labs</span>
            </div>
            <p style="font-size: 0.95rem; color: #888; margin-top: 15px;">
                Stay tuned for updates on this course.
            </p>
            <a href="#newsletter" class="resource-cta secondary">Get Notified</a>
        </div>
        <!-- Placeholder for Future Course 3 -->
        <div class="resource-card" style="opacity: 0.7; border-style: dashed;">
            <span class="resource-badge coming-soon">Coming Soon</span>
            <span class="resource-icon">🤖</span>
            <h3>SD-WAN Automation with Python</h3>
            <p>Automate SD-WAN deployments and management using Python, REST APIs, and modern DevOps practices.</p>
            <div class="resource-meta">
                <span>⏱️ TBD</span>
                <span>💻 Video + Code</span>
            </div>
            <p style="font-size: 0.95rem; color: #888; margin-top: 15px;">
                Stay tuned for updates on this course.
            </p>
            <a href="#newsletter" class="resource-cta secondary">Get Notified</a>
        </div>
    </div>
</div>

<!-- PDF Guides Section -->
<div class="resource-section">
    <h2>📄 PDF Guides & Resources</h2>
    <p style="color: #666; font-size: 1.05rem; margin-bottom: 30px;">
        Detailed technical guides and reference materials. Download instantly and keep forever.
    </p>
    <div class="pdf-list">
        <!-- Paid PDF Example 1 -->
        <div class="pdf-item">
            <span class="pdf-badge">$12</span>
            <h4>SD-WAN Deployment Checklist</h4>
            <p>Complete step-by-step checklist for planning and deploying Cisco SD-WAN in enterprise environments. Includes pre-deployment planning, configuration templates, and post-deployment validation.</p>    
            <div class="pdf-meta">
                <span>📄 15 pages</span>
                <span>✓ Templates included</span>
            </div>
            <a href="/checkout/sdwan-deployment-checklist" class="pdf-download">
                Get PDF — $12
            </a>
        </div>
        <!-- Free PDF Example -->
        <div class="pdf-item">
            <span class="pdf-badge free">Free</span>
            <h4>UX 2.0 Quick Reference</h4>
            <p>One-page cheat sheet covering Configuration Groups and Policy Groups basics. Perfect for keeping on your desk during migrations.</p>
            <div class="pdf-meta">
                <span>📄 1 page</span>
                <span>🎯 Quick reference</span>
            </div>
            <a href="/pdfs/ux2-quick-reference.pdf" class="pdf-download" download>
                Download Free
            </a>
        </div>
        <!-- Paid PDF Example 2 -->
        <div class="pdf-item">
            <span class="pdf-badge">$9</span>
            <h4>Common SD-WAN Issues & Fixes</h4>
            <p>Troubleshooting guide covering the top 10 SD-WAN problems network engineers encounter. Each issue includes symptoms, root causes, and step-by-step fixes.</p>
            <div class="pdf-meta">
                <span>📄 12 pages</span>
                <span>🔧 10 solutions</span>
            </div>
            <a href="/checkout/sdwan-troubleshooting-guide" class="pdf-download">
                Get PDF — $9
            </a>
        </div>
        <!-- Paid PDF Example 3 -->
        <div class="pdf-item">
            <span class="pdf-badge">$15</span>
            <h4>SD-WAN Security Best Practices</h4>
            <p>Comprehensive security guide covering firewall policies, IPsec encryption, segmentation strategies, and compliance requirements for enterprise SD-WAN deployments.</p>
            <div class="pdf-meta">
                <span>📄 22 pages</span>
                <span>🔒 Security focused</span>
            </div>
            <a href="/checkout/sdwan-security-guide" class="pdf-download">
                Get PDF — $15
            </a>
        </div>
        <!-- Add more PDFs as you create them -->
    </div>
    <p style="text-align: center; margin-top: 40px; color: #888;">
        <em>More guides coming soon. <a href="#newsletter" style="color: #0078D4;">Subscribe</a> to get notified when new resources are available.</em>
    </p>
</div>

<!-- Lab Files Section (Optional) -->
<div class="resource-section">
    <h2>🧪 Lab Files & Configs</h2>
    <div class="resource-grid">
        <div class="resource-card">
            <span class="resource-badge free">Free</span>
            <span class="resource-icon">⚙️</span>
            <h3>Basic SD-WAN Lab Topology</h3>
            <p>Ready-to-use lab setup with 2 vEdges, vManage, vBond, and vSmart. Perfect for getting started.</p>
            <div class="resource-meta">
                <span>📦 CML Ready</span>
                <span>🔧 Running Configs</span>
            </div>
            <a href="/labs/basic-sdwan-lab.zip" class="resource-cta" download>Download Lab Files</a>
        </div>
        <!-- Placeholder for more labs -->
        <div class="resource-card" style="opacity: 0.7; border-style: dashed;">
            <span class="resource-badge coming-soon">Coming Soon</span>
            <span class="resource-icon">🏗️</span>
            <h3>Multi-Tenant SD-WAN Lab</h3>
            <p>Advanced lab setup demonstrating multi-tenancy, VRF segmentation, and policy isolation.</p>
            <a href="#newsletter" class="resource-cta secondary">Get Notified</a>
        </div>
    </div>
</div>

<!-- Newsletter CTA Section -->
<div id="newsletter" style="background: linear-gradient(135deg, #0078D4 0%, #005A9E 100%); padding: 50px 30px; border-radius: 8px; text-align: center; margin-top: 60px; color: white;">
    <h2 style="color: white; font-size: 2rem; margin-bottom: 20px;">Stay Updated</h2>
    <p style="font-size: 1.2rem; margin-bottom: 30px; opacity: 0.95;">
        Get notified when new courses, guides, and resources are released. No spam, just valuable SD-WAN content.
    </p>
    <a href="/ux2-course#waitlist" style="display: inline-block; background: white; color: #0078D4; padding: 15px 35px; border-radius: 6px; text-decoration: none; font-weight: 600; font-size: 1.1rem;">
        Join the Waitlist
    </a>
</div>

</div>