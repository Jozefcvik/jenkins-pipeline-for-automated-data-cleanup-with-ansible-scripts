# Jenkins Pipeline for Automated Data Cleanup

This repository contains a Jenkins pipeline that automates the cleanup of old files and directories on target hosts using Ansible. The pipeline is designed to execute three sequential Ansible playbooks, ensuring structured logging, retention management, and notifications for deleted items.

##

### 📌 Features

- **Automated cleanup of old data:** Deletes files older than a configured retention period (default: 14 days) from a specified parent directory.
- **Multi-stage Ansible execution:**
  1. delete_old_data_001.yml — Discover subdirectories and initialize logging.
  2. delete_old_data_002.yml — Process each subdirectory, marking status as RUN, DONE, or FAILED.
  3. delete_old_data_003.yml — Scan and delete old files, remove empty directories, and maintain detailed logs.
- **Detailed logging:**
  - deleted_items_<build_number>.log — Tracks deleted files.
  - scanned_directories_<build_number>.log — Tracks directories processed and their status.
  - all_data_<build_number>.log — Complete snapshot of scanned paths with timestamps.
- **Retention management:** Ensures only data older than the specified retention period is deleted.
- **Email notifications:** Sends a summary email including logs and deletion details after each build.
- **Timeout safety:** Configurable pipeline timeout (default: 6 days) to prevent indefinitely running jobs.
- **Workspace cleanup:** Cleans Jenkins workspace after job completion to save disk space.

## 

### ⚙️ Jenkins Pipeline Overview

```groovy
pipeline {
    agent any
    options {
        timeout(time: 6, unit: 'DAYS')
    }
    environment {
        LOG_DIR = "/var/log/jenkins/ansible-logs/data_cleanup_job"
    }
    stages {
        stage('Execute playbook') {
            steps {
                sh '''
                    cd /etc/ansible/data_cleanup_job
                    export ANSIBLE_CONFIG=../ansible.cfg
                    ansible-playbook cleanup_task.yml -e "build_number=${BUILD_NUMBER}"
                    mkdir -p "$WORKSPACE/logs"
                    cp "$LOG_DIR"/deleted_items_${BUILD_NUMBER}.log "$WORKSPACE/logs/" || true
                    cp "$LOG_DIR"/scanned_directories_${BUILD_NUMBER}.log "$WORKSPACE/logs/" || true
                '''
            }
        }
    }
    post {
        always {
            // Send email notification and clean workspace
        }
    }
}
```

##

### 📝 Ansible Playbooks

1️⃣ **delete_old_data_001.yml**
- Checks if the parent directory exists.
- Initializes log files on the controller.
- Discovers subdirectories for further processing.
- Ensures logs are not empty if no data is found.

2️⃣ **delete_old_data_002.yml**
- Iterates through each discovered subdirectory.
- Updates log status to RUN, DONE, or FAILED.
- Invokes delete_old_data_003.yml for actual cleanup tasks.

3️⃣ **delete_old_data_003.yml**
- Deletes files older than the retention period.
- Deletes empty directories, including the target directory if empty.
- Tracks deleted files in a dedicated log.
- Fetches logs back to the controller for aggregation.

##

### 📂 Log Management

|Log File	                                  | Description                                                                  |
|-------------------------------------------|------------------------------------------------------------------------------|
|deleted_items_<build_number>.log	          | Files deleted during the cleanup                                             |
|scanned_directories_<build_number>.log	    | Directories scanned and their processing status                              |
|all_data_<build_number>.log	              | Complete list of all files and directories with last-activity timestamps     |

##

### ✉️ Notifications

After each pipeline run, an email is sent to notify stakeholders with:
- Job name and build number.
- Build URL.
- Deletion timestamp.
- Deleted items summary.
- Optional attachments: deleted items and scanned directories logs.

##

### ⚡ Requirements

- **Jenkins** with pipeline support.
- **Ansible** installed on the Jenkins agent.
- SSH connectivity to target_hosts.
- Proper permissions to delete files and directories on target hosts.
- Workspace and log directory accessible to Jenkins.

##

### 🔧 Usage
1. Copy the repository to the Jenkins workspace.
2. Configure ANSIBLE_CONFIG and LOG_DIR.
3. Trigger the pipeline with the build number.
4. Monitor logs in the Jenkins workspace under logs/.
5. Receive email notification after completion.

##

### 📌 Notes
- Ensure that parent_dir exists on target hosts, otherwise the playbooks will log a specific message and skip execution.
- Retention period is configurable but defaults to 14 days.
- The pipeline is safe against missing directories and empty data sets.


