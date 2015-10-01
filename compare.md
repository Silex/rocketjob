---
layout: default
---

### Compare [rocketjob][0] features with [rocketjob-pro][2], Resque, and Sidekiq

<table id="compare">
  <tr>
    <th>Feature</th>
    <th>rocketjob</th>
    <th>rocketjob-pro</th>
    <th><a href="https://github.com/resque/resque">Resque</a></th>
    <th><a href="http://sidekiq.org">Sidekiq</a></th>
  </tr>
  <tr>
    <td>Reliability</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>Job specific priority based processing</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Cron Replacement</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>Job Retention</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Collect Job Output</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Web UI</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
  </tr>
  <tr>
    <td>Individual Job Status</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>Future Dated Jobs</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
  </tr>
  <tr>
    <td>Directory Monitor</td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Batch Jobs</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>Pause & Resume Batch Jobs</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Encryption</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Compression</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Large file support</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Read and write Zip files</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Read and write GZip files</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Read XLSX files</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
  </tr>
  <tr>
    <td>Rate Limiting</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>Dedicated Support</td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
  <tr>
    <td>License</td>
    <td align="center">GPL</td>
    <td align="center">Commercial</td>
    <td align="center">MIT</td>
    <td align="center">LGPL / Commercial</td>
  </tr>
  <tr>
    <td>Pricing</td>
    <td align="center">Free</td>
    <td align="center">$75 / month /</br>100 worker threads</td>
    <td align="center">Free</td>
    <td align="center">Free / Commercial</td>
  </tr>
</table>

### Key

<table id="Key">
  <tr>
    <th>Yes</th>
    <th>No</th>
    <th>Requires Add-on</br>or limited</th>
  </tr>
  <tr>
    <td align="center"><img src="images/yes.png" alt="Y"></td>
    <td align="center"><img src="images/no.png" alt="N"></td>
    <td align="center"><img src="images/info.png" alt="?"></td>
  </tr>
</table>


### [Next: Directory Monitor ==>](dirmon.html)

[0]: http://rocketjob.io
[1]: https://github.com/rocketjob/rocketjob_mission_control
[2]: pro.html
[3]: http://sidekiq.org
[4]: http://sidekiq.org
