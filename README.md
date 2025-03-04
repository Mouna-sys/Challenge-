import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

@SpringBootApplication
public class EnergyApp {

    public static void main(Stringargs) {
        SpringApplication.run(EnergyApp.class, args);
    }
}

@Entity
class EnergyData {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String device;
    private double energyUsage;
    private LocalDateTime timestamp;

    // Constructors, getters, and setters
    public EnergyData() {}

    public EnergyData(String device, double energyUsage, LocalDateTime timestamp) {
        this.device = device;
        this.energyUsage = energyUsage;
        this.timestamp = timestamp;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getDevice() {
        return device;
    }

    public void setDevice(String device) {
        this.device = device;
    }

    public double getEnergyUsage() {
        return energyUsage;
    }

    public void setEnergyUsage(double energyUsage) {
        this.energyUsage = energyUsage;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(LocalDateTime timestamp) {
        this.timestamp = timestamp;
    }
}

interface EnergyDataRepository extends JpaRepository<EnergyData, Long> {
}

@RestController
class EnergyController {

    private final EnergyDataRepository energyDataRepository;
    private final Random random = new Random();

    public EnergyController(EnergyDataRepository energyDataRepository) {
        this.energyDataRepository = energyDataRepository;
    }

    @GetMapping("/energy")
    public List<EnergyData> getEnergyData() {
        simulateEnergyData();
        return energyDataRepository.findAll();
    }

    private void simulateEnergyData() {
        Stringdevices = {"Refrigerator", "Oven", "Washing Machine", "Lights"};
        String device = devices[random.nextInt(devices.length)];
        double energyUsage = 0.1 + (5.0 - 0.1) * random.nextDouble();
        LocalDateTime timestamp = LocalDateTime.now();
        energyDataRepository.save(new EnergyData(device, energyUsage, timestamp));
    }
}

// Front-End (HTML and JavaScript)
// index.html
<!DOCTYPE html>
<html>
<head>
    <title>Smart Home Energy Snapshot</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Energy Dashboard</h1>
    <canvas id="energyChart"></canvas>
    <script>
        async function fetchEnergyData() {
            const response = await fetch('/energy'); // Ensure this matches your spring boot port
            const data = await response.json();
            return data;
        }

        async function createChart() {
            const energyData = await fetchEnergyData();
            const labels = energyData.map(item => item.device);
            const usage = energyData.map(item => item.energyUsage);

            const ctx = document.getElementById('energyChart').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Energy Usage',
                        data: usage,
                        backgroundColor: 'rgba(54, 162, 235, 0.5)',
                        borderColor: 'rgba(54, 162, 235, 1)',
                        borderWidth: 1
                    }]
                },
                options: {
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });
        }

        createChart();
    </script>
</body>
</html>
