---
layout: default
title: TaskTrackerAPI
---

<div class="max-w-3xl space-y-10">

<section>
  <h2 class="text-2xl font-semibold mb-4">Problem</h2>

  <p class="text-gray-700">
    Manual task tracking and deployment workflows were inconsistent, error-prone,
    and difficult to reproduce across environments.
  </p>

  <ul class="list-disc pl-6 mt-4 text-gray-700">
    <li>No standard deployment pipeline</li>
    <li>Manual EC2 setup</li>
    <li>No automated testing or security checks</li>
  </ul>
</section>

<section>
  <h2 class="text-2xl font-semibold mb-4">Architecture</h2>

  <h3 class="text-lg font-medium mt-6">Backend</h3>
  <ul class="list-disc pl-6 mt-2 text-gray-700">
    <li>FastAPI service</li>
    <li>RESTful API design</li>
    <li>Pydantic-based request validation</li>
  </ul>

  <h3 class="text-lg font-medium mt-6">Infrastructure</h3>
  <ul class="list-disc pl-6 mt-2 text-gray-700">
    <li>AWS EC2, S3, RDS</li>
    <li>Terraform for Infrastructure as Code</li>
    <li>Docker for containerisation</li>
  </ul>

  <h3 class="text-lg font-medium mt-6">CI/CD</h3>
  <ul class="list-disc pl-6 mt-2 text-gray-700">
    <li>GitLab CI/CD pipelines</li>
    <li>Automated test execution</li>
    <li>Image scanning and controlled deployment</li>
  </ul>

  <div class="mt-6 bg-gray-100 rounded-md p-4 font-mono text-sm">
    Code → CI → Docker → AWS EC2
  </div>
</section>

<section>
  <h2 class="text-2xl font-semibold mb-4">Outcome</h2>

  <ul class="list-disc pl-6 text-gray-700">
    <li>Standardised CI/CD workflow from commit to deployment</li>
    <li>Repeatable, idempotent infrastructure provisioning</li>
    <li>Reduced deployment friction and runtime failures</li>
    <li>Production-ready service exposed via public endpoint</li>
  </ul>
</section>

</div>
