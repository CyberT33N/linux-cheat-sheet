# VeraCrypt Cleanup From Electron (v2)

Second-generation cleanup used in the Electron main process (see `electron.js`): dismounts all VeraCrypt containers, then removes stale `/dev/mapper` entries through a single `pkexec` call behind a native admin-rights dialog.

```javascript

// Function to cleanup VeraCrypt mounts and device mapper entries
async function cleanupVeraCrypt() {
    if (cleanupPromise) return cleanupPromise // Return existing cleanup promise if one is in progress
    if (isCleaningUp) return Promise.resolve() // Shouldn't happen, but just in case
    
    isCleaningUp = true
    cleanupPromise = (async() => {
        try {
            // First try to dismount all VeraCrypt volumes
            execSync('veracrypt --text --dismount --non-interactive', { stdio: 'pipe' })
            console.log('Successfully dismounted all VeraCrypt containers')

            // Check for stale device mapper entries
            const mapperLs = execSync('ls -l /dev/mapper', { stdio: 'pipe' }).toString()
            const veracryptEntries = mapperLs
                .split('\n')
                .filter(line => line.includes('veracrypt'))
                .map(line => {
                    // Split by spaces but keep multiple spaces as one separator
                    const parts = line.trim().split(/\s+/)
                    // The name is the third-to-last part before " -> ../dm-X"
                    return parts[parts.length - 3]
                })
                .filter(entry => entry && entry.startsWith('veracrypt'))

            if (veracryptEntries.length > 0) {
                console.log('Found stale VeraCrypt device mapper entries:', veracryptEntries)
                
                // Show native dialog box
                await dialog.showMessageBox(mainWindow, {
                    type: 'warning',
                    title: '⚠️ Administrator Rights Required',
                    message: '🔐 Administrator privileges are required to clean up VeraCrypt entries',
                    buttons: ['Continue'],
                    defaultId: 0,
                    noLink: true
                })

                // Combine all cleanup commands into a single pkexec call
                const cleanupCommands = veracryptEntries.map(entry => {
                    return [
                        `dmsetup remove "${entry}" 2>/dev/null || dmsetup remove -f "${entry}" 2>/dev/null || true`,
                        `umount -f "/dev/mapper/${entry}" 2>/dev/null || true`,
                        `dmsetup remove -f "${entry}" 2>/dev/null || true`
                    ].join(' && ')
                }).join(' && ')

                if (cleanupCommands) {
                    try {
                        execSync(`pkexec /bin/bash -c "${cleanupCommands}"`, { stdio: 'pipe' })
                        console.log('Successfully cleaned up stale device mapper entries')
                    } catch (err) {
                        console.error('Failed to remove device mapper entries:', err.message)
                    }
                }
            }
        } catch (error) {
            console.error('Error during VeraCrypt cleanup:', error.message)
        } finally {
            isCleaningUp = false
            cleanupPromise = null
        }
    })()
    
    return cleanupPromise
}
```