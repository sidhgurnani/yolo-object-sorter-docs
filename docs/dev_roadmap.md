# Relative Development Timeline

This timeline shows the relative development timeline of the two most important aspects of the project: software and physical prototype. The goal here is to emphasize that this is a parallel process rather than working on developing software fully first then the physical prototype. Click on each card to go to the page with more information about that specific version along the development path.

<div class="dev-timeline-wrapper">
  <!-- Header -->
  <div class="dev-timeline-header">
    <div class="dev-header-left">
      <h2>Software</h2>
    </div>
    <div class="dev-header-center"></div>
    <div class="dev-header-right">
      <h2>Physical Prototype</h2>
    </div>
  </div>

  <!-- Beginning Marker -->
  <div class="dev-timeline-item">
    <div class="dev-item-left"></div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-marker dev-marker-start"></div>
      <span class="dev-label">Beginning of Development (June 2025)</span>
    </div>
    <div class="dev-item-right"></div>
  </div>

  <!-- Software V0 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left">
      <div class="dev-card dev-card-software">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/WebUSB_Findings/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 0</h3>
            <p>Attempted to build a web app and communicate information between app and Arduino via WebUSB. Got basic communication to work, but ended up having issues getting webcam to work.</p>
        </a>
      </div>
    </div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-left"></div>
    </div>
    <div class="dev-item-right"></div>
  </div>

  <!-- Software V1 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left">
      <div class="dev-card dev-card-software">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/software_v1/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 1</h3>
            <p>Switched to Python App and programmed a graphical user interface (GUI) using tkinter to allow user to perform basic functions. Basic features included, but not all requirements met as outlined in Milestone 3.</p>
        </a>
      </div>
    </div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-left"></div>
    </div>
    <div class="dev-item-right"></div>
  </div>

  <!-- Physical V1 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left"></div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-right"></div>
    </div>
    <div class="dev-item-right">
      <div class="dev-card dev-card-hardware">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/vibration_bowl_feeder/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 1</h3>
            <p>Attempted to implement a vibration bowl feeder, and went through two design iterations before shelving the idea due to increased overall complexity for a part of the project that was mostly insignificant.</p>
        </a>
      </div>
    </div>
  </div>

  <!-- Software V2 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left">
      <div class="dev-card dev-card-software">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/software_v2/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 2</h3>
            <p>Refined Python App for more modern look by opting to use customtkinter and fully implemented sorting franework. Minimum viable software achieved at this stage.</p>
        </a>
      </div>
    </div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-left"></div>
    </div>
    <div class="dev-item-right"></div>
  </div>

  <!-- Physical V2 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left"></div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-right"></div>
    </div>
    <div class="dev-item-right">
      <div class="dev-card dev-card-hardware">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/prototype_v2/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 2</h3>
            <p>Changed development path and prototyped a standalone device with an external USB webcam and a servo-controlled platform to sort objects into bins.</p>
        </a>
      </div>
    </div>
  </div>

  <!-- Software V3 -->
  <div class="dev-timeline-item">
    <div class="dev-item-left">
      <div class="dev-card dev-card-software">
        <a href="https://sidhgurnani.github.io/yolo-object-sorter-docs/software_v3/" style="text-decoration: none; color: inherit; display: block;">
            <h3>Version 3</h3>
            <p>Further refined version of software, focused on user experience, interface, and control. Implemented an even more modern appearance and focused on simplifying controls to make it more intuitive for end user.</p>
        </a>
      </div>
    </div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-connector dev-connector-left"></div>
    </div>
    <div class="dev-item-right"></div>
  </div>

  <!-- End Marker -->
  <div class="dev-timeline-item">
    <div class="dev-item-left"></div>
    <div class="dev-item-center">
      <div class="dev-line"></div>
      <div class="dev-marker dev-marker-end"></div>
      <span class="dev-label">End of Development / Final Integration (November 2025)</span>
    </div>
    <div class="dev-item-right"></div>
  </div>
</div>

<style>
  .dev-timeline-wrapper {
    max-width: 1400px !important;
    margin: 40px auto !important;
    padding: 20px !important;
    clear: both !important;
  }

  .dev-timeline-header {
    display: table !important;
    width: 100% !important;
    margin-bottom: 40px !important;
    table-layout: fixed !important;
  }

  .dev-header-left,
  .dev-header-center,
  .dev-header-right {
    display: table-cell !important;
    vertical-align: middle !important;
  }

  .dev-header-left,
  .dev-header-right {
    width: 45% !important;
  }

  .dev-header-center {
    width: 10% !important;
  }

  .dev-header-left h2,
  .dev-header-right h2 {
    text-align: center !important;
    margin: 0 !important;
    padding: 15px !important;
    border-radius: 4px !important;
    font-size: 1.3rem !important;
    background: var(--md-code-bg-color, #f5f5f5) !important;
    color: var(--md-default-fg-color, #000) !important;
    border: 1px solid var(--md-default-fg-color--lightest, #e0e0e0) !important;
  }

  .dev-header-left h2 {
    border-left: 4px solid var(--md-primary-fg-color, #4051b5) !important;
  }

  .dev-header-right h2 {
    border-left: 4px solid var(--md-accent-fg-color, #526cfe) !important;
  }

  .dev-timeline-item {
    display: table !important;
    width: 100% !important;
    margin-bottom: 30px !important;
    table-layout: fixed !important;
    position: relative !important;
  }

  .dev-item-left,
  .dev-item-center,
  .dev-item-right {
    display: table-cell !important;
    vertical-align: middle !important;
  }

  .dev-item-left,
  .dev-item-right {
    width: 45% !important;
    padding: 0 20px !important;
  }

  .dev-item-center {
    width: 10% !important;
    position: relative !important;
    text-align: center !important;
  }

  .dev-line {
    position: absolute !important;
    left: 50% !important;
    top: 0 !important;
    bottom: 0 !important;
    width: 3px !important;
    background: var(--md-default-fg-color--lightest, #e0e0e0) !important;
    transform: translateX(-50%) !important;
  }

  .dev-connector {
    position: absolute !important;
    top: 50% !important;
    height: 3px !important;
    width: 50% !important;
    background: var(--md-default-fg-color--lightest, #e0e0e0) !important;
    transform: translateY(-50%) !important;
    z-index: 1 !important;
  }

  .dev-connector-left {
    right: 50% !important;
  }

  .dev-connector-right {
    left: 50% !important;
  }

  .dev-marker {
    position: absolute !important;
    left: 50% !important;
    top: 50% !important;
    transform: translate(-50%, -50%) !important;
    width: 16px !important;
    height: 16px !important;
    border-radius: 50% !important;
    background: var(--md-default-bg-color, #fff) !important;
    border: 3px solid var(--md-default-fg-color--light, #999) !important;
    z-index: 2 !important;
  }

  .dev-marker-start {
    background: var(--md-primary-fg-color, #4051b5) !important;
    border-color: var(--md-primary-fg-color, #4051b5) !important;
  }

  .dev-marker-end {
    background: var(--md-accent-fg-color, #526cfe) !important;
    border-color: var(--md-accent-fg-color, #526cfe) !important;
  }

  .dev-label {
    position: absolute !important;
    left: 110% !important;
    top: 50% !important;
    transform: translateY(-50%) !important;
    background: var(--md-code-bg-color, #f5f5f5) !important;
    color: var(--md-default-fg-color, #000) !important;
    padding: 6px 12px !important;
    border-radius: 4px !important;
    border: 1px solid var(--md-default-fg-color--lightest, #e0e0e0) !important;
    font-weight: 600 !important;
    font-size: 0.85rem !important;
    white-space: nowrap !important;
    z-index: 2 !important;
  }

  .dev-card {
    padding: 16px !important;
    border-radius: 4px !important;
    background: var(--md-code-bg-color, #f5f5f5) !important;
    border: 1px solid var(--md-default-fg-color--lightest, #e0e0e0) !important;
    transition: transform 0.2s, box-shadow 0.2s !important;
  }

  .dev-card:hover {
    transform: translateY(-2px) !important;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1) !important;
  }

  .dev-card-software {
    border-left: 3px solid var(--md-primary-fg-color, #4051b5) !important;
  }

  .dev-card-hardware {
    border-left: 3px solid var(--md-accent-fg-color, #526cfe) !important;
  }

  .dev-card h3 {
    margin-top: 0 !important;
    margin-bottom: 10px !important;
    font-size: 1.1rem !important;
    color: var(--md-default-fg-color, #000) !important;
  }

  .dev-card p {
    margin: 0 !important;
    color: var(--md-default-fg-color--light, #666) !important;
    line-height: 1.6 !important;
    font-size: 0.9rem !important;
  }

  @media (max-width: 768px) {
    .dev-timeline-header,
    .dev-timeline-item {
      display: block !important;
    }

    .dev-header-left,
    .dev-header-center,
    .dev-header-right,
    .dev-item-left,
    .dev-item-center,
    .dev-item-right {
      display: block !important;
      width: 100% !important;
    }

    .dev-item-center {
      display: none !important;
    }

    .dev-card {
      margin-bottom: 20px !important;
    }

    .dev-label {
      position: static !important;
      display: block !important;
      text-align: center !important;
      margin: 20px 0 !important;
      transform: none !important;
    }
  }
</style>