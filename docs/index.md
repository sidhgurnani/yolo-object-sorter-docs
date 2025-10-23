# YOLO-Based Binary Object Sorting System

Welcome! This GitHub page will contain documentation regarding what this project is about as well as progress for where I am at and all developments that have occured. This will serve as a log of my progress.

I just want to give a special thank you to Dr. Nirav Merchant from the University of Arizona for being an incredible mentor throughout the whole project. 

Check out my [engineering portfolio](https://sidhgurnani.github.io/sidh-ME-portfolio/) to see other projects!

## About Me

My name is Sidh Gurnani, and I studied Mechanical Engineering at Purdue University, and graduated with my Bachelor's in May 2025. I have been working on this project since June 2024.

## Project Status

<!-- 🚀 Project Status Dashboard Block -->
<div class="project-status">
  <h2>🤖 YOLO Binary Object Sorter</h2>

  <div class="progress-ring">
    <svg>
      <circle class="bg" r="50" cx="60" cy="60"></circle>
      <circle class="progress" r="50" cx="60" cy="60"></circle>
    </svg>
    <span id="progress-text">0%</span>
  </div>

  <ul class="milestones">
    <li class="done">✅ Milestone 1: Refresh programming knowledge</li>
    <li class="done">✅ Milestone 2: Understand Basics of Machine Learning</li>
    <li class="done">✅ Milestone 3: Develop Basic Software</li>
    <li class="done"> ✅ Milestone 4: 3D Model and Design Physical Product</li>
    <li class="current">⭕ Milestone 5: Integrate Final Software and Hardware</li>
    <li>⭕ Milestone 6: Final Testing</li>
    <li>⭕ Milestone 7: Reflection and Next Steps</li>
  </ul>

  <div class="next">
    📌 <strong>Up Next:</strong> Final Software Bug Fixes
  </div>
</div>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');

.project-status {
  background: #1e293b;
  color: #e2e8f0;
  font-family: 'Inter', sans-serif;
  border-radius: 16px;
  padding: 2rem;
  width: 100%;
  max-width: 100%; /* allow full content width */
  margin: 1.5rem 0;
  box-shadow: 0 0 20px rgba(0,0,0,0.4);
  text-align: center;
  box-sizing: border-box;
}

/* Optional: add spacing consistency inside mkdocs-material content area */
.md-content .project-status {
  margin-left: auto;
  margin-right: auto;
  width: 100%;
}

.project-status h2 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #38bdf8;
}

/* Progress Ring */
.progress-ring {
  position: relative;
  width: 120px;
  height: 120px;
  margin: 1.5rem auto;
}

.progress-ring svg {
  transform: rotate(-90deg);
  width: 120px;
  height: 120px;
}

.progress-ring circle {
  fill: none;
  stroke-width: 10;
  r: 50;
  cx: 60;
  cy: 60;
}

.progress-ring .bg {
  stroke: #334155;
}

.progress-ring .progress {
  stroke: #38bdf8;
  stroke-linecap: round;
  transition: stroke-dashoffset 1s ease;
}

.progress-ring span {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.5rem;
  font-weight: bold;
}

/* Milestones */
.milestones {
  list-style: none;
  padding: 0;
  text-align: left;
  margin: 2rem 0;
}

.milestones li {
  margin: 0.5rem 0;
  padding: 0.5rem 1rem;
  border-radius: 10px;
  background: #0f172a;
  transition: transform 0.2s ease, background 0.3s ease;
}

.milestones li.done {
  background: rgba(34, 197, 94, 0.2);
  color: #22c55e;
}

.milestones li.current {
  background: rgba(250, 204, 21, 0.2);
  color: #facc15;
  box-shadow: 0 0 10px rgba(250, 204, 21, 0.5);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% { box-shadow: 0 0 5px rgba(250, 204, 21, 0.4); }
  50% { box-shadow: 0 0 15px rgba(250, 204, 21, 0.7); }
  100% { box-shadow: 0 0 5px rgba(250, 204, 21, 0.4); }
}

/* Next Milestone */
.next {
  margin-top: 1rem;
  padding: 1rem;
  background: rgba(56, 189, 248, 0.2);
  border: 1px solid #38bdf8;
  border-radius: 12px;
  font-size: 1rem;
  color: #38bdf8;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.4);
}

/* Responsive */
@media (max-width: 600px) {
  .project-status {
    padding: 1.2rem;
    max-width: 95%;
  }
  .progress-ring {
    width: 100px;
    height: 100px;
  }
}
</style>

<script>
document.addEventListener("DOMContentLoaded", function() {
  const milestones = document.querySelectorAll('.project-status .milestones li');
  const total = milestones.length;
  const completed = document.querySelectorAll('.project-status .milestones li.done').length;

  const percent = Math.round((completed / total) * 100);
  document.getElementById('progress-text').textContent = `${percent}%`;

  const circle = document.querySelector('.project-status .progress-ring .progress');
  const radius = circle.r.baseVal.value;
  const circumference = 2 * Math.PI * radius;

  circle.style.strokeDasharray = circumference;
  circle.style.strokeDashoffset = circumference;

  setTimeout(() => {
    const offset = circumference - (percent / 100) * circumference;
    circle.style.strokeDashoffset = offset;
  }, 300);
});
</script>