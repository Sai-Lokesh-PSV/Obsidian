```dataviewjs
const colorSleep = 'rgb(129, 177, 212)', uptimeColor = 'rgb(54, 162, 235)', bedtimeColor = 'rgb(255, 99, 132)';
// Stats
const statsPrecision = 2; // Number of decimal places to round to
const sleepAverageTimeWindow = 21, exerciseAverageTimeWindow = 14, uptimeAverageTimeWindow = 14,
    bedtimeAverageTimeWindow = 14, ratingAverageTimeWindow = 21;
// Streak thresholds
const exerciseStreakThreshold = 10, guitarStreakThreshold = 1, sleepStreakThreshold = 7.5;
// Targets
const targetBedtime = "23:30", targetUptime = "07:30", targetSleepDuration = 8, targetExerciseDuration = 30;
const dataTypes = [
    'sleep',
    'exercise',
    'guitar',
    'uptime',
    'bedtime',
    'rating',
]; // comment data is excluded from these visualizations; "reading" data exists until 2024-06-09
// From here onwards, I changed all the default values to -1, so there is no ambiguity in the data (e.g.: I don't know if I forgot to fill in the data or if I actually did nothing)
const dataChangeCutoffDate = "2023-11-14"

// # Collect data
const journalPath = '"Obsidian/daily-notes"';
const journals = dv.pages(journalPath).sort(j => j.file.name, 'asc');
const fileNames = journals.map(j => j.file.name);
const processedData = {};  // stores data and dataClean (no null values) for each dataType
for (let type of dataTypes) {
    processedData[type] = processData({journals: journals, dataType: type, cutoffDate: dataChangeCutoffDate});
}
const {
    sleep: {data: sleepData, dataClean: sleepDataClean},
    exercise: {data: exerciseData, dataClean: exerciseDataClean},
    guitar: {data: guitarData, dataClean: guitarDataClean},
    rating: {data: ratingData, dataClean: ratingDataClean},
    bedtime: {data: bedtimeData, dataClean: bedtimeDataClean},
    uptime: {data: uptimeData, dataClean: uptimeDataClean},
} = processedData;

function processData({journals, dataType, cutoffDate}) {
    const data = [], dataClean = [];
    for (let j of journals) {
        const value = j[dataType]
        let validValue = (value === 0 && j.file.name < cutoffDate) || value === -1 || value === undefined ? null : value;
        if ((dataType === 'bedtime' || dataType === 'uptime') && validValue !== null) {
            validValue = parseMilitaryTimeNumberToFraction(value);
        }
        data.push(validValue);
        if (validValue !== null) {
            dataClean.push(validValue);
        }
    }
    return {data, dataClean};
}

// # Rendering
renderStreakContainer();
dv.span("<br>");
window.renderChart(getWeeklySleepGoalsBarChart(sleepDataClean), this.container);
renderStatsTable({
    dataArrays: [sleepDataClean, exerciseDataClean, uptimeDataClean, bedtimeDataClean],
    titles: ["Sleep", "Exercise", "Uptime", "Bedtime"],
    units: ["h", "min", "", ""]
});
window.renderChart(getExerciseLineChart(), this.container);
window.renderChart(getRatingLineChart(ratingData, fileNames), this.container);
window.renderChart(getSleepLineChart(), this.container);
window.renderChart(getSleepHoursDensityChart(sleepDataClean), this.container);
window.renderChart(getBedtimeLineChart(), this.container);
window.renderChart(getSleepTimeDensityChart(uptimeDataClean, bedtimeDataClean), this.container);
// TODO find a way to export the charts to image files

// # Construct charts
// ## Line Charts
function getSleepLineChart() {
    return {
        type: 'line',
        data: {
            labels: fileNames,
            datasets: [
                {
                    label: 'Sleep Time (hours)',
                    data: sleepData,
                    borderColor: colorSleep,
                    borderWidth: 1,
                    pointRadius: 1,
                },
                {
                    label: 'EMA (window: ' + sleepAverageTimeWindow + 'd)',
                    data: exponentialMovingAverage(sleepData, sleepAverageTimeWindow),
                    borderColor: changeRgbAlpha('rgb(248, 252, 3)', 0.8),
                    backgroundColor: [],
                    borderWidth: 2.5,
                    borderDash: [5, 5], // Optional: make the line dashed
                    pointRadius: 0, // No points, just the line
                    fill: false, // No fill under the line
                },
            ]
        },
        options: {
            plugins: {
                annotation: {
                    annotations: [{
                        type: 'line',
                        yMin: 8,
                        yMax: 8,
                        borderColor: 'rgba(255, 99, 132, 0.4)',
                        borderWidth: 2,
                        label: {
                            enabled: true,
                            content: 'Target: 8 hours',
                            position: 'end'
                        }
                    }]
                },
                tooltip: {
                    mode: 'index',
                }
            }
        }
    };
}

function getExerciseLineChart() {
    const offset = 1;  // offset to avoid log(0) in the log scale
    return {
        type: 'line',
        data: {
            labels: fileNames,
            datasets: [
                {
                    label: 'Exercise (minutes)',
                    data: exerciseData.map(value => value === null ? null : value + offset),
                    borderColor: "rgba(247, 170, 54, 0.9)",
                    borderWidth: 0.5,
                    pointRadius: 0.5,
                },
                {
                    label: 'EMA (window: ' + exerciseAverageTimeWindow + 'd)',
                    data: exponentialMovingAverage(exerciseData, exerciseAverageTimeWindow).map(value => value + offset),
                    borderColor: 'rgba(198, 129, 247, 0.9)',
                    borderWidth: 1,
                    borderDash: [5, 5],
                    pointRadius: 0.5,
                    fill: false,
                },
                {
                    label: 'Exercise Goal' + ` (${targetExerciseDuration} min)`,
                    data: Array(fileNames.length).fill(targetExerciseDuration),
                    borderColor: 'rgba(129, 247, 133, 0.6)',
                    borderWidth: 2,
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                    type: 'line'
                },
                {
                    label: 'Dedicated Exercise Threshold' + ` (${exerciseStreakThreshold} min)`,
                    data: Array(fileNames.length).fill(10),
                    borderColor: 'rgba(247, 129, 129, 0.6)',
                    borderWidth: 2,
                    borderDash: [5, 5],
                    pointRadius: 0,
                    fill: false,
                    type: 'line'
                }
            ],
        },
        options: {
            scales: {
                y: {
                    type: 'logarithmic',
                    beginAtZero: false,
                    title: {
                        display: true,
                        text: 'Exercise (minutes)'
                    }
                }
            },
            plugins: {
                tooltip: {
                    mode: 'index',
                }
            }
        }
    };
}

function getBedtimeLineChart() {
    const uptimeShifted = uptimeData.map(shiftTimeForVisualization),
        bedtimeShifted = bedtimeData.map(shiftTimeForVisualization);
    return {
        type: 'line',
        data: {
            labels: fileNames,
            datasets: [
                {
                    label: 'Wake Time',
                    data: uptimeShifted,
                    borderColor: uptimeColor,
                    fill: false,
                    pointRadius: 1,
                    borderWidth: 1,
                },
                {
                    label: 'Bed Time',
                    data: bedtimeShifted,
                    borderColor: bedtimeColor,
                    fill: false,
                    pointRadius: 1,
                    borderWidth: 1,
                },
                {
                    label: `EMA Wake Time (window: ${uptimeAverageTimeWindow}d)`,
                    data: exponentialMovingAverage(uptimeShifted, uptimeAverageTimeWindow),
                    borderColor: 'rgba(92, 242, 162, 1)',
                    fill: false,
                    pointRadius: 0,
                    borderWidth: 1,
                    steppedLine: 'before',
                },
                {
                    label: `SMA Wake Time (window: ${uptimeAverageTimeWindow}d)`,
                    data: simpleMovingAverage(uptimeShifted, uptimeAverageTimeWindow),
                    borderColor: 'rgba(92, 242, 162, 0.6)',
                    fill: false,
                    pointRadius: 0,
                    borderWidth: 1,
                    steppedLine: 'before',
                },
                {
                    label: `EMA Bed Time (window: ${bedtimeAverageTimeWindow}d)`,
                    data: exponentialMovingAverage(bedtimeShifted, bedtimeAverageTimeWindow),
                    borderColor: 'rgba(255, 206, 86, 1)',
                    fill: false,
                    pointRadius: 0,
                    borderWidth: 1,
                    steppedLine: 'before',
                },
                {
                    label: `SMA Bed Time (window: ${bedtimeAverageTimeWindow}d)`,
                    data: simpleMovingAverage(bedtimeShifted, bedtimeAverageTimeWindow),
                    borderColor: 'rgba(255, 206, 86, 0.6)',
                    fill: false,
                    pointRadius: 0,
                    borderWidth: 1,
                    steppedLine: 'before',
                },
                {
                    label: `Target Uptime (${targetUptime})`,
                    data: new Array(fileNames.length).fill(shiftTimeForVisualization(parseMilitaryTimeStringToFraction(targetUptime))),
                    borderColor: 'rgb(153, 102, 255)',
                    borderWidth: 2,
                    pointRadius: 0,
                    fill: false,
                    borderDash: [5, 5]
                },
                {
                    label: `Target Bedtime (${targetBedtime})`,
                    data: new Array(fileNames.length).fill(shiftTimeForVisualization(parseMilitaryTimeStringToFraction(targetBedtime))),
                    borderColor: 'rgb(75, 192, 192)',
                    borderWidth: 2,
                    pointRadius: 0,
                    fill: false,
                    borderDash: [5, 5]
                },
            ]
        },
        options: {
            scales: {
                y: {
                    type: 'linear',
                    position: 'left',
                    min: 18,
                    max: 42, // 24 + 18 to accommodate the shifted time range
                    ticks: {
                        callback: function (value, index, values) {
                            let displayHour = value % 24;
                            return displayHour + ':00'; // Formatting y-axis labels as conventional time
                        }
                    },
                    title: {
                        display: true,
                        text: 'Time of Day'
                    }
                },
                x: {
                    title: {
                        display: true,
                        text: 'Date'
                    }
                }
            },
            plugins: {
                tooltip: {
                    mode: "index",
                    callbacks: {
                        label: function (context) {
                            let label = '';
                            let value = context.raw;
                            let hour = Math.floor(value);
                            let minute = Math.round((value - hour) * 60);
                            let displayHour = hour % 24;
                            label += `${displayHour}:${minute < 10 ? '0' + minute : minute}`;
                            return label;
                        }
                    },
                },
            }
        }
    };
}

function getRatingLineChart(ratingData, fileNames) {
    return {
        type: 'line',
        data: {
            labels: fileNames,
            datasets: [
                {
                    label: 'Daily Rating',
                    data: ratingData,
                    borderColor: 'rgba(75, 192, 192, 0.5)',
                    borderWidth: 1,
                    pointRadius: 0,
                },
                {
                    label: 'Average Rating',
                    data: exponentialMovingAverage(ratingData, ratingAverageTimeWindow),
                    borderColor: 'rgba(255, 99, 132, 0.5)',
                    borderWidth: 1,
                    pointRadius: 0,
                    steppedLine: 'before',
                }
            ]
        },
        options: {
            scales: {
                y: {
                    reverse: true,
                    min: 1,
                    max: 5,
                    stepSize: 1,
                    title: {
                        display: true,
                        text: 'Rating (1 = best, 5 = worst)'
                    }
                }
            },
            plugins: {
                tooltip: {
                    mode: 'index',
                }
            }
        }
    };
}

// ## Bar Charts
function getWeeklySleepGoalsBarChart(cleanSleepData) {
    // historyLength ...  historyLength buckets of 7 days
    const historyLength = 4;
    const weeklySleepTotals = [...Array(historyLength)].map((_, i) => {
        const start = cleanSleepData.length - (i + 1) * 7;
        const end = start + 7;
        return cleanSleepData.slice(start, end).reduce((a, b) => a + b, 0);
    });
    const goal = 60;
    const colorThresholds = [
        {threshold: 47, color: '#c41244'}, // < 47 -> Dark Red
        {threshold: 49, color: '#f54242'}, // < 49 -> Red
        {threshold: 53, color: '#f57542'}, // < 52 -> Orange
        {threshold: 56, color: '#bfbb47'}, // < 54 -> Yellowish green
        {threshold: goal, color: '#7bbf47'}, // < goal -> Greenish
        {threshold: 66, color: '#42f554'}, // < 66 Bright green 
        {threshold: Infinity, color: '#f5a742'}, // > 66 -> Orange
    ];
    return {
        type: 'bar',
        data: {
            labels: [...Array(historyLength)].map((_, i) => `Past ${i * 7} - ${i * 7 + 7} days`),
            datasets: [
                {
                    label: 'Total Sleep Hours',
                    data: weeklySleepTotals,
                    backgroundColor: weeklySleepTotals.map((total) => {
                        for (let i = 0; i < colorThresholds.length; i++) {
                            if (total < colorThresholds[i].threshold) {
                                return colorThresholds[i].color;
                            }
                        }
                    }),
                }
            ]
        },
        options: {
            aspectRatio: 2.75,
            scales: {
                y: {
                    min: 38,
                    title: {
                        display: true,
                        text: 'Hours'
                    }
                }
            },
            plugins: {
                legend: {
                    display: false // Disable default legend
                },
                title: {
                    display: true,
                    text: 'Weekly Sleep Hours vs Goals'
                },
                tooltip: {
                    callbacks: {
                        label: function (context) {
                            let label = '';
                            let value = context.raw;
                            let belowGoal = goal - value;
                            if (belowGoal > 0) {
                                label += `${value} total sleep hours. ${belowGoal.toFixed(2)} hours below goal of ${goal}`;
                            } else if (belowGoal < 0) {
                                label += `${value} total sleep hours. ${Math.abs(belowGoal).toFixed(2)} hours above goal of ${goal}`;
                            } else {
                                label += `${value} total sleep hours. Goal of ${goal} hours met`;
                            }
                            return label;
                        }
                    }
                },
                annotation: {
                    annotations: [
                        {
                            type: 'line',
                            yMin: 56,
                            yMax: 56,
                            borderColor: 'rgb(255, 99, 132)',
                            borderWidth: 2,
                            label: {
                                enabled: true,
                                content: 'Goal: 60 hours',
                                position: 'end'
                            },
                        }
                    ]
                }
            }
        }
    };
}

// ## Density Charts
function getSleepTimeDensityChart(uptimes, bedtimes) {
    function createBinnedData(data, binSize, startHour) {
        let numberOfBins = Math.ceil((24 - startHour) / binSize) + Math.ceil(startHour / binSize);
        let binnedData = Array(numberOfBins).fill(0);
        data.forEach(time => {
            let adjustedTime = time - startHour;
            if (adjustedTime < 0) adjustedTime += 24; // Wrap around for negative times
            let binIndex = Math.floor(adjustedTime / binSize);
            if (binIndex >= 0 && binIndex < binnedData.length) {
                binnedData[binIndex]++;
            }
        });
        return binnedData;
    }

    const binSize = 0.5; // in hours
    const startHour = 5; // Start the bins from 5:00 AM
    let bedTimesBinned = createBinnedData(bedtimes, binSize, startHour);
    let upTimesBinned = createBinnedData(uptimes, binSize, startHour);

    function formatTime(hour) {
        let h = Math.floor(hour);
        let m = Math.round((hour - h) * 60);
        return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}`;
    }

    const sleepDensityChartConfig = {
        type: 'bar',
        data: {
            labels: [...Array(bedTimesBinned.length).keys()].map(x => formatTime((x * binSize + startHour) % 24)),
            datasets: [
                {
                    label: 'Bed Time Density',
                    data: bedTimesBinned,
                    backgroundColor: 'rgba(255, 99, 132, 0.5)',
                    borderColor: bedtimeColor,
                    borderWidth: 1
                },
                {
                    label: 'Wake-up Time Density',
                    data: upTimesBinned,
                    backgroundColor: 'rgba(54, 162, 235, 0.5)',
                    borderColor: uptimeColor,
                    borderWidth: 1
                }
            ]
        },
        options: {
            scales: {
                y: {
                    beginAtZero: true,
                    title: {
                        display: true,
                        text: 'Frequency'
                    }
                },
                x: {
                    title: {
                        display: true,
                        text: 'Time of Day'
                    }
                }
            },
            plugins: {
                tooltip: {
                    callbacks: {
                        label: function (context) {
                            let label = context.dataset.label;
                            let index = context.dataIndex;
                            let start = formatTime((index * binSize + startHour) % 24);
                            let end = formatTime(((index + 1) * binSize + startHour) % 24);
                            return `${label}: ${context.raw} entries from ${start} to ${end}`;
                        }
                    }
                }
            }
        }
    };
    return sleepDensityChartConfig;
}

function getSleepHoursDensityChart(sleepDurations) {
    function createHistogram(data, binSize) {
        let maxDuration = Math.ceil(Math.max(...data));
        let numberOfBins = Math.ceil(maxDuration / binSize);
        let histogram = Array(numberOfBins).fill(0);
        data.forEach(duration => {
            let binIndex = Math.floor(duration / binSize);
            if (binIndex < histogram.length) {
                histogram[binIndex]++;
            }
        });
        return histogram;
    }

    const binSize = 0.5; // Bins of 0.5 hours
    let histogramData = createHistogram(sleepDurations, binSize);
    const sleepHoursDensityChartConfig = {
        type: 'bar',
        data: {
            labels: [...Array(histogramData.length).keys()].map(x => `${(x * binSize).toFixed(1)}`),
            datasets: [
                {
                    label: 'Sleep Duration Density',
                    data: histogramData,
                    backgroundColor: changeRgbAlpha(colorSleep, 0.8),
                    borderWidth: 1
                }
            ]
        },
        options: {
            scales: {
                y: {
                    beginAtZero: true,
                    title: {
                        display: true,
                        text: 'Frequency'
                    }
                },
                x: {
                    title: {
                        display: true,
                        text: 'Sleep Duration (hours)'
                    }
                }
            },
            plugins: {
                tooltip: {
                    callbacks: {
                        label: function (context) {
                            return `${context.dataset.label}: ${context.raw} occurrences for ${context.label} hrs`;
                        }
                    }
                }
            }
        }
    };
    return sleepHoursDensityChartConfig;
}

// # Utility functions

function changeRgbAlpha(rgb, alpha) {
    const [red, green, blue] = rgb.match(/rgb\((\d+),\s*(\d+),\s*(\d+)\)/).slice(1).map(Number);
    return `rgba(${red}, ${green}, ${blue}, ${alpha})`;
}

function parseMilitaryTimeStringToFraction(timeStr) {
    // e.g. "23:30" -> 23.5
    if (!timeStr) return null;
    let [hours, minutes] = timeStr.split(':').map(Number);
    return hours + minutes / 60;
}

function parseMilitaryTimeNumberToFraction(militaryTimeNumber) {
    // e.g. 2330 -> 23.5
    if (!militaryTimeNumber) return null;
    const time = parseInt(militaryTimeNumber);
    const hours = Math.floor(time / 100);
    const minutes = time % 100;
    return hours + minutes / 60;
}

function shiftTimeForVisualization(time) {
    const bedtimeChartTimeShift = 18; // Shift times before 18:00 to the next day
    return time === null ? null : time < bedtimeChartTimeShift ? time + 24 : time;
}

function exponentialMovingAverage(data, timeWindow) {
    // ema fills in the missing data with the previous value -> at sections where the data is missing, the ema will be constant
    const alpha = 2 / (timeWindow + 1);
    return data.reduce((ema, value, index) => {
        if (value !== null) {
            ema.push(index === 0 ? value : alpha * value + (1 - alpha) * ema[index - 1]);
        } else if (index !== 0) {
            ema.push(ema[index - 1]);
        } else {
            ema.push(null);
        }
        return ema;
    }, []);
}

function simpleMovingAverage(data, windowSize) {
    return data.map((_, i) => {
        const window = data.slice(Math.max(0, i - windowSize + 1), i + 1);
        const lastValidValue = window.slice().reverse().find(value => value !== null);
        if (lastValidValue === undefined) {
            // If all values in the window are null, return null
            return null;
        }
        const validValues = window.filter(value => value !== null);
        const sum = validValues.reduce((sum, value) => sum + value, 0);
        const count = validValues.length;
        return (sum + (window.length - count) * lastValidValue) / window.length;
    });
}


// # Statistics
function renderStatsTable({dataArrays, titles, units}) {
    let meanRow = ['Mean'], medianRow = ['Median'], stdDevRow = ['Standard Deviation'],
        errorRow = ['Difference to Goal'];
    let tableHeader = ['Statistics'];
    titles.forEach(title => tableHeader.push(title));
    dataArrays.forEach((data, index) => {
        const title = titles[index]
        const isClockTime = (title == "Bedtime" || title == "Uptime")
        let mean = isClockTime ? calculateAdjustedMeanTime(data) : calculateMean(data);
        let median = isClockTime ? calculateMedian(data) : calculateMedian(data);
        const variance = (title == "Bedtime" || title == "Uptime") ? calculateCircularVariance(data, mean) : calculateVariance(data, mean);
        const stdDev = Math.sqrt(variance).toFixed(statsPrecision);

        let error = null
        if (isClockTime) {
            error = calculateTimeError(mean, title === "Bedtime" ? targetBedtime : targetUptime, title === "Bedtime" ? 1 : -1);
        } else if (title === "Sleep") {
            error = (targetSleepDuration - mean).toFixed(statsPrecision)
        } else if (title === "Exercise") {
            error = (targetExerciseDuration - mean).toFixed(statsPrecision)
        }

        mean = isClockTime ? timeToMilitary(mean) : mean.toFixed(statsPrecision)
        median = isClockTime ? timeToMilitary(median) : median.toFixed(statsPrecision)
        meanRow.push(`${mean} ${units[index]}`);
        medianRow.push(`${median} ${units[index]}`);
        stdDevRow.push(`${stdDev} ${units[index]}`);
        errorRow.push(`${error} ${units[index]}`);
    });
    dv.table(tableHeader, [meanRow, medianRow, stdDevRow, errorRow]);
}

function calculateMean(data) {
    return data.reduce((a, b) => a + b, 0) / data.length;
}

function calculateMedian(data) {
    const mid = Math.floor(data.length / 2), nums = [...data].sort((a, b) => a - b);
    return data.length % 2 !== 0 ? nums[mid] : (nums[mid - 1] + nums[mid]) / 2;
}

function calculateVariance(data, mean) {
    return data.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / data.length;
}

function calculateCircularVariance(data, mean) {
    const sqDiffs = data.map(x => {
        let diff = Math.abs(x - mean);
        if (diff > 12) diff = 24 - diff; // 12 hours = halfway around the clock
        return diff ** 2;
    });
    return sqDiffs.reduce((a, b) => a + b, 0) / data.length;
}

function calculateAdjustedMeanTime(times) {  // time is special, since it jumps areound (i.e. the mean of 23 and 1 should be 0)
    let mean = calculateMean(adjustTimesForSleepTimeMeanCalculation(times));
    return mean < 0 ? 24 + mean : mean;  // Adjust back if negative
}

function adjustTimesForSleepTimeMeanCalculation(times) {
    // Shift times: If time is after 18:00, consider it as early next day for mean calculation
    return times.map(time => time >= 18 ? time - 24 : time);
}

function timeToMilitary(time) {
    let hours = Math.floor(time);
    let minutes = Math.round((time - hours) * 60);
    return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;
}

// meanTime: fractional, targetTime: HH:MM, uptime: -1 if uptime else 1
function calculateTimeError(meanTime, targetTime, uptime) {
    let meanHour = Math.floor(meanTime), meanMinute = Math.round((meanTime - meanHour) * 60);
    let targetHour = parseInt(targetTime.split(':')[0]), targetMinute = parseInt(targetTime.split(':')[1]);
    let meanTotalMinutes = meanHour * 60 + meanMinute;
    let targetTotalMinutes = targetHour * 60 + targetMinute;
    let errorInMinutes = meanTotalMinutes - targetTotalMinutes;
    let sign = errorInMinutes * uptime < 0 ? -1 : 1;
    errorInMinutes = Math.abs(errorInMinutes);
    if (errorInMinutes > 720) {
        errorInMinutes = 1440 - errorInMinutes;
    } // 720 minutes = 12 hours, 1440 minutes in a day
    let errorHours = Math.floor(errorInMinutes / 60);
    let errorMinutes = errorInMinutes % 60;
    return `${errorHours * sign} h ${errorMinutes} min`;
}

// # Streaks
function renderStreakContainer() {
    const guitarStreakInfo = calculateStreak({
        data: guitarData,
        fileNames: fileNames,
        valueThreshold: guitarStreakThreshold
    }), exerciseStreakInfo = calculateStreak({
        data: exerciseData,
        fileNames: fileNames,
        valueThreshold: exerciseStreakThreshold
    }), sleepStreakInfo = calculateStreak({
        data: sleepData,
        fileNames: fileNames,
        valueThreshold: sleepStreakThreshold
    });

    function renderStreakDisplay({streakInfo, streakEmoji, title, threshold, unit}) {
        const streakCard = dv.el("div", "", {
            cls: "streak-card",
            attr: {title: `Minimum value to maintain streak: ${threshold} ${unit}`}
        });
        dv.el("h2", title, {container: streakCard, cls: "streak-header"});
        dv.el("p", `**Streak**: ${streakInfo.currentStreak} days ${streakEmoji}`, {
            container: streakCard,
            cls: "streak-info"
        });
        dv.el("p", `**Past week**: ${streakInfo.last7StreakCount}/7 days`, {container: streakCard, cls: "streak-info"});
        dv.el("p", `**Max**: ${streakInfo.maxStreak} days`, {container: streakCard, cls: "streak-info"});
        return streakCard;
    }

    const streaksContainer = dv.el("div", "", {cls: "streak-container"}),
        exerciseStreakCard = renderStreakDisplay({
            streakInfo: exerciseStreakInfo,
            streakEmoji: streakEmojiIndicator(exerciseStreakInfo.currentStreak),
            title: "🏋️",
            threshold: exerciseStreakThreshold,
            unit: "minutes"
        }), guitarStreakCard = renderStreakDisplay({
            streakInfo: guitarStreakInfo,
            streakEmoji: streakEmojiIndicator(guitarStreakInfo.currentStreak),
            title: "🎸",
            threshold: guitarStreakThreshold,
            unit: "minutes"
        }), sleepStreakCard = renderStreakDisplay({
            streakInfo: sleepStreakInfo,
            streakEmoji: streakEmojiIndicator(sleepStreakInfo.currentStreak),
            title: "😴",
            threshold: sleepStreakThreshold,
            unit: "hours"
        });
    streaksContainer.appendChild(exerciseStreakCard);
    streaksContainer.appendChild(guitarStreakCard);
    streaksContainer.appendChild(sleepStreakCard);
    return streaksContainer;
}

// Streak Calculation (if a daily not is missing, it doesn't count towards the streak)
// There is a joker for missing one day, where the streak is not increased, but also not reset
function calculateStreak({data, fileNames, valueThreshold}) {
    let currentStreak = 0, streakJokerAvailable = true, maxStreak = 0;
    const isStreakDay = (value) => value !== null && value !== undefined && value >= valueThreshold;
    for (let i = 0; i < data.length; i++) {
        const currentDate = parseDateFromFilename(fileNames[i]), isStreak = isStreakDay(data[i]);
        if (isStreak) {
            currentStreak++;
        } else if (streakJokerAvailable && currentStreak > 0) {
            streakJokerAvailable = false;
            continue;
        } else {
            // Reset streak if data is missing or invalid
            currentStreak = 0;
            streakJokerAvailable = true;
        }
        maxStreak = Math.max(maxStreak, currentStreak);
    }
    const last7StreakCount = data.slice(-7).filter(isStreakDay).length;
    return {currentStreak, maxStreak, last7StreakCount};
}

function parseDateFromFilename(filename) {
    // Assuming filename format is 'YYYY-MM-DD'
    const parts = filename.split("-");
    return new Date(parts[0], parts[1] - 1, parts[2]);
}

function streakEmojiIndicator(streak) {
    if (streak >= 100) return "🌟🌟🌟"; // God level
    if (streak == 69) return "😎"; // Nice level
    if (streak >= 56) return "🎇🌟"; // Legendary level
    if (streak >= 49) return "🌌💫"; // Cosmic level
    if (streak == 42) return "👽"; // Answer to the Ultimate Question of Life, the Universe, and Everything
    if (streak >= 35) return "🚀🌕"; // Lunar level
    if (streak >= 28) return "🎸🎶"; // Rockstar level
    if (streak >= 21) return "🏆🎉"; // Champion level
    if (streak >= 14) return "🔥🌟"; // On fire
    if (streak >= 10) return "🌟✨"; // Starry progress
    if (streak >= 7) return "🔥💪"; // Powering through
    if (streak >= 5) return "🎯🔥"; // On target
    if (streak >= 3) return "🌱🌿"; // Growing strong
    if (streak > 0) return "⚙️"; // Working on it
    return "🔄"; // Reset
}
Heatmaps
dv.span("**🎸 Guitar 🎸**")

const calendarDataTemplate = {
    colors: {
        pink: ["#ff96cb", "#ff70b8", "#ff3a9d", "#ee0077", "#c30062"],
    },
    entries: []
};

let currentYear = new Date().getFullYear();
let calendarDataByYear = {};

for(let page of dv.pages('"Obsidian/daily-notes"')){
    if (page.guitar > 0) {
        let entry = {
            date: page.file.name,
            intensity: page.guitar,
            content: await dv.span(`[[${page.guitar}|]]`)
        };
        let entryYear = new Date(page.file.name).getFullYear();
        if (!calendarDataByYear[entryYear]) {
            calendarDataByYear[entryYear] = {...calendarDataTemplate, year: entryYear, entries: []};
        }
        calendarDataByYear[entryYear].entries.push(entry);
    }
}

function renderHeatmapForYear(year) {
    let heatmapContainer = dv.el("div", "", {cls: "heatmap-container"});
    if (calendarDataByYear[year]) {
        renderHeatmapCalendar(heatmapContainer, calendarDataByYear[year]);
    } else {
        heatmapContainer.innerHTML = `<p>No data for year ${year}</p>`;
    }
    dv.container.replaceChild(heatmapContainer, dv.container.querySelector('.heatmap-container'));
}

function createYearButtons(years) {
    const buttonContainer = dv.el("div", "", {cls: "year-button-container"});
    years.forEach(year => {
        const button = dv.el("button", `${year}`, {
            cls: "year-button",
            attr: {type: "button"}
        });
        button.addEventListener("click", () => {
            renderHeatmapForYear(year);
        });
        buttonContainer.appendChild(button);
    });
    return buttonContainer;
}

// Get unique years from the data
let years = Object.keys(calendarDataByYear).sort((a, b) => b - a); // Sort years in descending order

// Render year buttons and the initial heatmap
dv.container.appendChild(createYearButtons(years));
dv.container.appendChild(dv.el("div", "", {cls: "heatmap-container"})); // Placeholder for the heatmap
renderHeatmapForYear(currentYear);

// Inject styles
const style = document.createElement('style');
style.textContent = `
    .year-button-container {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
    }
    .year-button {
        padding: 10px;
        background-color: #4CAF50;
        color: white;
        border: none;
        cursor: pointer;
    }
    .year-button:hover {
        background-color: #45a049;
    }
    .heatmap-container {
        margin-top: 20px;
    }
`;
document.head.appendChild(style);
```

