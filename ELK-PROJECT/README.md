<section id="elk-overview">
  <div align="center" style="margin-bottom:12px;">
    <img width="1211" height="393" alt="Beats → Logstash → Elasticsearch → Kibana pipeline"
         src="https://github.com/user-attachments/assets/cbf044c9-3ded-4a3a-8062-01fcc0f0c207" />
    <div style="font-size:12px; color:#666;">Beats → Logstash → Elasticsearch → Kibana (ELK)</div>
  </div>

  <h2>ELK Overview</h2>
  <p>
    <strong>ELK</strong> stands for <strong>Elasticsearch</strong>, <strong>Logstash</strong>, and <strong>Kibana</strong>. 
    It’s a log and telemetry analytics stack used to collect events, parse and enrich them, store and search at scale, and visualize the results for investigations.
  </p>

<h3>What ELK Is Typically Used For</h3>
  <ul>
    <li><strong>Centralized logging & search</strong> across endpoints, servers, applications, and network devices.</li>
    <li><strong>Security operations</strong>: alert triage, threat hunting, IOC/behavioral pivoting, and incident investigation.</li>
    <li><strong>Dashboards & visualization</strong> to monitor authentication trends, error spikes, and service health.</li>
    <li><strong>Compliance & audit</strong>: retaining and querying audit trails (Windows Event Logs, proxy/firewall logs).</li>
    <li><strong>Application & performance monitoring</strong> using metrics and traces alongside logs for faster root-cause analysis.</li>
  </ul>
   
<section id="scenario">
  <h2>Scenario</h2>
  <p>
    During normal SOC monitoring, Analyst John observed an alert on an IDS solution indicating a potential C2 communication from a user Browne from the HR department. A suspicious file was accessed containing a malicious pattern THM:{ ________ }. A week-long HTTP connection logs have been pulled to investigate. Due to limited resources, only the connection logs could be pulled out and are ingested into the connection_logs index in Kibana.
  </p>
  <p>
    Our task in this room will be to examine the network connection logs of this user, find the link and the content of the file, and answer the questions.
  </p>
</section>

</section>
<hr style="margin:24px 0;">
<section id="next-section">
  <p>
   I went straight to <strong>Discover</strong> because it’s the quickest way to see raw events in
    <code>connection_logs</code>. From there I can set the time to March 2022 and get the exact
    document count instantly...no dashboards or saved searches needed. It’s a fast sanity check before any deeper pivots.
  </p>
  <div align="center" style="margin-top:8px;">
    <img width="247" height="441" alt="Kibana Discover view — March 2022 event count" src="https://github.com/user-attachments/assets/e5aa196c-c804-4f3b-8f52-f4b243ef5333" />
  </div>
</section>

</section>
<hr style="margin:24px 0;">
<section id="next-section">
  
<section id="q1-answer">
  <h3>Question 1 — Total events in March 2022</h3>
  <p><strong>Answer:</strong> <strong>1482 events</strong>.</p>

  <h4>How I got it (quick walkthrough)</h4>
  <ol>
    <li>Opened <strong>Discover</strong> and selected the <code>connection_logs</code> data view.</li>
    <li>Set an absolute time range for March:
      <br/><code>2022-03-01 00:00:00</code> → <code>2022-03-31 00:00:00</code> (end is exclusive).</li>
    <li>Confirmed no extra filters, then hit <strong>Refresh</strong>.</li>
    <li>Read the top document count (“Hits”) → <strong>1482</strong>.</li>
  </ol>
    <div align="center" style="margin-top:8px;">
    <img width="2096" height="520" alt="Kibana Discover — March 2022 event count" src="https://github.com/user-attachments/assets/1450cd59-0348-4634-b2a2-8a58ccfe88b3" />
  </div>

  <h4>Quick checks</h4>
  <ul>
    <li>Time zone is correct to avoid off-by-one issues.</li>
    <li>Sampling is off so the count reflects all documents.</li>
  </ul>
  <div align="center" style="margin-top:8px;">
    <img width="596" height="45" alt="Kibana time range selector — March 2022" src="https://github.com/user-attachments/assets/fdb0da2f-4eac-4619-af0e-1f900f009d5d" />
  </div>
</section>


</section>
<hr style="margin:24px 0;">
<section id="next-section">
  
<section id="q2-answer">
  <h3>Question 2 — Suspected user’s IP address</h3>
  
<h4>How I got it</h4>
<ol>
    <li>Stayed in <strong>Discover</strong> on the <code>connection_logs</code> view with March 2022 timeboxed.</li>
    <li><strong>Filter applied:</strong> The search pill <code>source_ip: 192.166.65.54</code> isolates the suspected host within the March 2022 window.</li>
    <li><strong>Hits & timeline:</strong> The histogram shows <strong>2 hits</strong> around Mar 10...light activity that fits low-and-slow C2 behavior.</li>
    <li><strong>Table view:</strong> Columns include <code>source_ip</code> and <code>user_agent</code>; both rows show <code>bitsadmin</code>, used to quietly download files.</li>
    <li><strong>Next pivot:</strong> From here, I added <code>destination.ip</code> / <code>url</code> to trace where the host connected (used in later questions).</li>
   <div align="center" style="margin-top:8px;">
    <img width="1619" height="464" alt="Kibana Discover filtered on source_ip 192.166.65.54" src="https://github.com/user-attachments/assets/47d17d02-a894-436f-8d36-5b229a969b0d" />
  </div>


