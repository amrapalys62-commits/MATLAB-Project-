# MATLAB-Project-
MATLAB Project Details
%% Interactive Student Performance Analysis Simulator (MATLAB CLI Loop)
clear; clc; close all;

% Default simulation parameters
n          = 80;
slopeHours = 4.2;   % Points gained per weekly study hour
groupShift = 4.5;   % Mean score offset for Group B vs Group A
noiseSD    = 5.0;   % Random noise standard deviation
seedVal    = 42;

while true
    rng(seedVal);

    % 1. Generate student dataset
    studyHours = max(0.5, min(12, round(normrnd(5.2, 2.1, [n, 1]), 1)));
    attendance = max(45,  min(100, round(normrnd(82, 11, [n, 1]), 1)));
    group      = categorical(repmat({'Group A'; 'Group B'}, n/2, 1));
    shiftVec   = (group == 'Group B') * groupShift;
    noise      = normrnd(0, noiseSD, [n, 1]);
    finalScore = max(0, min(100, 32 + slopeHours * studyHours + 0.28 * attendance + shiftVec + noise));

    T = table(studyHours, attendance, group, finalScore);

    % 2. Descriptive Statistics
    clc;
    fprintf('=== CURRENT PARAMETERS ===\n');
    fprintf('n = %d | Study Slope = %.2f pts/hr | Group B Shift = %.2f pts | Noise SD = %.2f | Seed = %d\n\n', ...
        n, slopeHours, groupShift, noiseSD, seedVal);

    fprintf('=== 1. DESCRIPTIVE STATISTICS ===\n');
    vars = {'studyHours', 'attendance', 'finalScore'};
    for k = 1:numel(vars)
        x = T.(vars{k});
        fprintf('%-12s | Mean: %6.2f | Median: %6.2f | SD: %6.2f | IQR: %6.2f | Min-Max: [%.1f, %.1f]\n', ...
            vars{k}, mean(x), median(x), std(x), iqr(x), min(x), max(x));
    end

    % 3. Linear Regression
    fprintf('\n=== 2. LINEAR REGRESSION (finalScore ~ studyHours + attendance) ===\n');
    mdl = fitlm(T, 'finalScore ~ studyHours + attendance');
    disp(mdl.Coefficients);
    fprintf('R-squared: %.4f | Adjusted R-squared: %.4f\n', ...
        mdl.Rsquared.Ordinary, mdl.Rsquared.Adjusted);

    mdlSimple = fitlm(T, 'finalScore ~ studyHours');
    b0 = mdlSimple.Coefficients.Estimate(1);
    b1 = mdlSimple.Coefficients.Estimate(2);

    % 4. Two-Sample t-Test
    fprintf('\n=== 3. TWO-SAMPLE T-TEST (Group A vs. Group B) ===\n');
    scoresA = T.finalScore(T.group == 'Group A');
    scoresB = T.finalScore(T.group == 'Group B');
    [h, pVal, ci, stats] = ttest2(scoresA, scoresB);
    fprintf('Group A Mean: %.2f (SD %.2f) | Group B Mean: %.2f (SD %.2f)\n', ...
        mean(scoresA), std(scoresA), mean(scoresB), std(scoresB));
    fprintf('t = %.4f, df = %d, p = %.4f, 95%% CI = [%.2f, %.2f], Reject H0: %d\n', ...
        stats.tstat, stats.df, pVal, ci(1), ci(2), h);

    % 5. Update Figure
    fig = findobj('Type', 'figure', 'Name', 'Interactive Student Performance Simulator');
    if isempty(fig)
        fig = figure('Name', 'Interactive Student Performance Simulator', 'Position', [100, 100, 1150, 400]);
    else
        figure(fig);
        clf;
    end

    subplot(1, 2, 1);
    scatter(T.studyHours, T.finalScore, 36, 'filled'); hold on;
    xGrid = linspace(min(T.studyHours), max(T.studyHours), 100)';
    plot(xGrid, b0 + b1 * xGrid, 'r-', 'LineWidth', 2);
    xlabel('Weekly Study Hours');
    ylabel('Final Score (%)');
    title(sprintf('OLS Fit: Score = %.2f + %.2f*Hours (R^2 = %.2f)', b0, b1, mdlSimple.Rsquared.Ordinary));
    grid on;

    subplot(1, 2, 2);
    boxchart(T.group, T.finalScore);
    xlabel('Group');
    ylabel('Final Score (%)');
    title(sprintf('Group Comparison (p = %.4f)', pVal));
    grid on;

    % 6. Interactive Menu
    fprintf('\nChoose an action:\n');
    fprintf('  [1] Change Study Hour Slope\n');
    fprintf('  [2] Change Group B Mean Shift\n');
    fprintf('  [3] Change Noise SD\n');
    fprintf('  [4] Change Random Seed\n');
    fprintf('  [5] Exit Simulator\n');
    choice = input('Enter option (1-5): ');

    switch choice
        case 1
            val = input('New Study Hour Slope (e.g., 0 to 10): ');
            if isnumeric(val) && isscalar(val) && ~isnan(val), slopeHours = val; end
        case 2
            val = input('New Group B Shift (e.g., -10 to 15): ');
            if isnumeric(val) && isscalar(val) && ~isnan(val), groupShift = val; end
        case 3
            val = input('New Noise SD (e.g., 1 to 15): ');
            if isnumeric(val) && isscalar(val) && ~isnan(val) && val > 0, noiseSD = val; end
        case 4
            val = input('New Random Seed integer (e.g., 100): ');
            if isnumeric(val) && isscalar(val) && ~isnan(val), seedVal = round(val); end
        case 5
            fprintf('Exiting simulator.\n');
            break;
        otherwise
            fprintf('Invalid option; re-running with current parameters.\n');
    end
end
