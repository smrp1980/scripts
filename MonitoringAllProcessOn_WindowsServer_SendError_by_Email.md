# Here's an equivalent script written in Windows PowerShell to monitor running processes,
#check for zombie processes or processes with high CPU usage, and send an email notification if any issues are found:
#-------------------------------------------------------------------------------------------------------------------------#

#Body of Script
# Define recipient email address
$recipient = "recipient@example.com"

# Function to send an email
function Send-Email {
    param(
        [string]$subject,
        [string]$body
    )

    Send-MailMessage -To $recipient -Subject $subject -Body $body -SmtpServer "smtp.example.com"
}

# Function to check if a process is a zombie
function Is-ProcessZombie {
    param(
        [int]$pid
    )

    $process = Get-Process -Id $pid -ErrorAction SilentlyContinue
    if ($process -and $process.Threads.Count -eq 0) {
        return $true
    }

    return $false
}

# Function to check if a process exceeds CPU threshold
function Is-ProcessHighCPU {
    param(
        [int]$pid,
        [int]$cpuThreshold
    )

    $process = Get-Process -Id $pid -ErrorAction SilentlyContinue
    if ($process -and $process.CPU -gt $cpuThreshold) {
        return $true
    }

    return $false
}

# Main script

# CPU threshold in percentage
$cpuThreshold = 90

# Get all running processes
$processes = Get-Process

# Loop through each process
foreach ($process in $processes) {
    # Check if the process is a zombie
    if (Is-ProcessZombie -pid $process.Id) {
        $subject = "Zombie Process Alert"
        $body = "Process with PID $($process.Id) is in a zombie state."
        Send-Email -subject $subject -body $body
    }

    # Check if the process exceeds CPU threshold
    if (Is-ProcessHighCPU -pid $process.Id -cpuThreshold $cpuThreshold) {
        $subject = "High CPU Usage Alert"
        $body = "Process '$($process.ProcessName)' with PID $($process.Id) is consuming high CPU time."
        Send-Email -subject $subject -body $body
    }
}



