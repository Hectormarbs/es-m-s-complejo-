
/**
 * HIGH-PERFORMANCE SYSTEM MONITOR
 
 */

const os = require('os');

class SystemAnalyzer {
    constructor(updateInterval = 1000) {
        this.updateInterval = updateInterval;
        this.metricsHistory = [];
    }

    // Calcula la carga de CPU de forma asíncrona
    async getCPUUsage() {
        const stats1 = os.cpus();
        await new Promise(resolve => setTimeout(resolve, 500));
        const stats2 = os.cpus();

        return stats1.map((cpu, i) => {
            const idle = stats2[i].times.idle - cpu.times.idle;
            const total = Object.values(stats2[i].times).reduce((a, b) => a + b) -
                          Object.values(cpu.times).reduce((a, b) => a + b);
            return ((1 - idle / total) * 100).toFixed(2);
        });
    }

    // Formatea bytes a unidades legibles (GB/MB)
    formatBytes(bytes) {
        return (bytes / 1024 / 1024 / 1024).toFixed(2) + ' GB';
    }

    async run() {
        console.clear();
        console.log('--- INICIANDO ANALIZADOR DE SISTEMA AVANZADO ---');

        setInterval(async () => {
            const cpuLoads = await this.getCPUUsage();
            const freeMem = os.freemem();
            const totalMem = os.totalmem();
            const usageMem = totalMem - freeMem;

            const snapshot = {
                timestamp: new Date().toLocaleTimeString(),
                cpu: cpuLoads,
                memory: ((usageMem / totalMem) * 100).toFixed(2)
            };

            this.updateUI(snapshot, totalMem, usageMem);
        }, this.updateInterval);
    }

    updateUI(data, total, used) {
        process.stdout.write('\x1Bc'); // Limpia la terminal sin parpadeo
        console.log(`[LOG] ${data.timestamp} | Memoria: ${data.memory}%`);
        console.log(`Utilizada: ${this.formatBytes(used)} / Total: ${this.formatBytes(total)}`);
        
        console.log('\nCarga por Núcleo:');
        data.cpu.forEach((load, idx) => {
            const bar = '█'.repeat(Math.floor(load / 5));
            console.log(`Core ${idx}: [${bar.padEnd(20, ' ')}] ${load}%`);
        });
    }
}

const monitor = new SystemAnalyzer();
monitor.run().catch(err => console.error('Error crítico:', err));
