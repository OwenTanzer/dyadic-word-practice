# Dyadic Word Practice · Agent guide

## Keep a work log the student can reopen

At the start of the session, look for the student’s existing work log and read the latest entry. If there is none, create class_work/work-log.md in their project, or use an editable document in their chosen workspace. Keep using the same file across sessions.

Append a dated entry at each checkpoint. Record the trial and source files, commands actually run, decisions about response units and boundaries, results inspected, errors and fixes, and paths to outputs. Distinguish student-reported results from checks you performed. Keep credentials and raw participant responses out of the log.

Before handoff, record the last verified checkpoint, unresolved questions and next concrete action. Save and reopen the file to verify the entry, then give the student its path or link. If you lack file-writing access, provide the entry as Markdown for the student to save and say that it still needs saving.

Help the student run the pipeline on their own computer. Ask which dataset, trial, operating system, repository folder and step they are working on. Start from their last verified checkpoint. Offer the next command and its expected result. Treat a checkmark as a learner report. Identify which outputs you inspected yourself. Stop at a failed check and record the command, working folder and exact error.

[Research repository](https://github.com/OwenTanzer/dyadic-word-generation). The lab login grants course access. Repository access and MATLAB installation are separate prerequisites.

## 1. Get ready

Before you begin

Complete the checklist below. You will run commands in MATLAB and your computer’s terminal.

Check MATLAB R2025b and Curve Fitting Toolbox, Git and Git LFS (Large File Storage), GitHub access to the project, an ordered response list and a plan for its similarity matrix.

Checkpoint: My tools, repository access, and data are ready.

## 2. Download the project

Git Bash · Windows / Terminal · Mac

Open Git Bash on Windows or Terminal on Mac. Run each line. Finish any GitHub sign-in prompt and wait for the download.

~~~sh
cd ~
git lfs install
git clone https://github.com/OwenTanzer/dyadic-word-generation.git
cd dyadic-word-generation
git lfs pull
~~~

Checkpoint: I can find Experiment 1 and analysis_code in the downloaded folder.

## 3. Open in MATLAB

MATLAB · Command Window

Click the Command Window. Run cd(uigetdir), select the repository folder, then run the remaining lines.

~~~matlab
cd(uigetdir)
% Select the dyadic-word-generation folder, then run:
addpath('analysis_code/diffusion_clustering')
mkdir('class_work')
dir
~~~

Checkpoint: Experiment 1 and analysis_code appear in the dir output.

## 4. See the human example

MATLAB · Command Window

Paste this block. A mean similarity curve should appear.

~~~matlab
R = load(['analysis_code/results/' ...
    'diffusionclustering_results.mat']);
plot(1:100, R.humans_all.lag(1:100), ...
    'LineWidth', 2)
xlabel('Lag in responses')
ylabel('Mean similarity')
~~~

Checkpoint: The saved human similarity curve is visible.

## 5. Follow the code

MATLAB · Editor

Open each file with the commands below. Find the highlighted terms in the callout. Leave the files unchanged, then return to the Command Window.

Locate humans_ in run_analysis.m, matrix_file in run_condition.m, the scale and seed settings in detect_clusters.m, and the diagonal means in lag_similarity.m.

~~~matlab
edit run_analysis.m
edit run_condition.m
edit detect_clusters.m
edit lag_similarity.m
~~~

Checkpoint: I found the condition, matrix loading, cluster settings, and diagonal means.

## 6. Run one human session

MATLAB · Command Window

Run the first block and wait for the >> prompt. Then plot and save the results.

~~~matlab
[one, summary] = run_condition(pwd, ...
    "Experiment 1", "humans_1168_paper", ...
    2, 40, 30);

one.response_count
plot(2:2:40, summary.wtd, 'o-', ...
    'LineWidth', 2)
xlabel('Filter size in responses')
ylabel('Weighted cluster rate')
save('class_work/human_session.mat', ...
    'one', 'summary')
exportgraphics(gcf, ...
    'class_work/human_clusters.png')
~~~

Checkpoint: The count is 1168 and human_clusters.png opens from class_work.

## 7. Prepare your list

MATLAB · Editor

Choose the new-dataset route or Theo’s original ZIP. Keep each participant and category in a separate file.

For a new dataset, place one UTF-8 text file per trial under class_work. Keep repeats and order. Remove headers and blank lines. For continuous prose, establish the response unit and trial boundaries with the researcher before converting. Keep a source-row audit. For Theo’s original ZIP, download [prepare_theo.m](https://tanzer.systems/ra-course/prepare_theo.m), place it beside the repository README, and run prepare_theo in MATLAB. Expected counts: theo_p1_food = 995, theo_p1_fruitveg = 234, theo_p2_food = 505, theo_p2_fruitveg = 204, theo_p3_food = 814, theo_p3_fruitveg = 191. Total: 2,943. [Theo folder](https://github.com/OwenTanzer/dyadic-word-generation/tree/main/outside_data/liam/theo_category_recall).

Checkpoint: My UTF-8 list has one response per line, with original order and repeats.

## 8. Match the matrix

Your project · class_work

Obtain a matching CSV from Owen and put it beside the TXT. Both names must match before the extension. The matrix must use this exact list in this exact order.

Prefer an existing matrix with verified sequence provenance. Optional embedding sends responses to OpenAI and incurs charges. Obtain Owen’s approval first. Use a private terminal prompt for the key. With Python 3.9+ in the repository folder, install openai and numpy, save this as embed_recall.py, then run python embed_recall.py (python3 on Mac). Preserve existing matrices before regeneration and discard partial output after failure.

~~~python
import getpass, os, subprocess, sys
os.environ['OPENAI_API_KEY'] = getpass.getpass(
    'OpenAI API key: ')
name = input('Trial name (no extension): ')
with open('class_work/' + name + '.txt',
          encoding='utf-8') as src, open(
          'class_work/' + name + '.csv', 'w',
          encoding='utf-8') as dst:
    subprocess.run([sys.executable,
        'analysis_code/preparation/embed.py'],
        stdin=src, stdout=dst, check=True)
~~~

Checkpoint: My TXT and CSV names match, and I have verified their sequence provenance.

## 9. Inspect your trial

MATLAB · Command Window

Choose the trial name without an extension. Paste the block. All three assertions must pass before you continue.

Replace recall_01 with the verified trial basename containing only letters, digits, hyphens or underscores. Run the remaining analysis blocks in the same MATLAB session.

~~~matlab
trial = "recall_01";
base = "class_work/" + trial;
S = readmatrix(base + ".csv");
words = readlines(base + ".txt");
words = words(~ismissing(words) & ...
    strlength(words) > 0);
n = numel(words);
assert(isequal(size(S), [n n]))
assert(all(isfinite(S), 'all'))
assert(n >= 7)
imagesc(S)
axis image
colorbar
xlabel('Response')
ylabel('Response')
~~~

Checkpoint: The matrix is n × n, finite, and has at least seven responses.

## 10. Measure the sequence

MATLAB · Command Window

Run this block after inspecting your trial. It saves a diffusion figure and displays the cluster-rate curve.

Compare trials only across shared lags and filter sizes. In individual recall, odd and even lags do not identify speakers.

~~~matlab
lags = 1:min(30, n-1);
similarity = arrayfun(@(k) ...
    mean(diag(S,k)), lags);
plot(lags, similarity, 'o-')
xlabel('Lag in responses')
ylabel('Mean similarity')
exportgraphics(gcf, ...
    base + "_diffusion.png")

smax = min(40, 2*floor((n-1)/6));
scales = 2:2:smax;
[peakWeights, intervalWeights, audit] = ...
    detect_clusters(S, 2, smax, 1:10);
rate = sum(peakWeights,2)'.*scales/n;
plot(scales, rate, 'o-')
xlabel('Filter size in responses')
ylabel('Weighted cluster rate')
~~~

Checkpoint: Both curves were produced without an error.

## 11. Save and hand off

MATLAB · Command Window

Save this trial’s variables, figures, and plot values. Open class_work and check the files before marking complete.

Keep the input pair, results, plots, exported values, original trial identity, conversion notes and matrix provenance together. For another trial, repeat steps 9–11.

~~~matlab
save(base + "_results.mat", ...
    'words', 'S', 'lags', 'similarity', ...
    'scales', 'rate', 'peakWeights', ...
    'intervalWeights', 'audit')
exportgraphics(gcf, ...
    base + "_clusters.png")
writetable(table(lags', similarity', ...
    'VariableNames', {'lag','similarity'}), ...
    base + "_diffusion.csv")
writetable(table(scales', rate', ...
    'VariableNames', {'scale','cluster_rate'}), ...
    base + "_clusters.csv")
~~~

Checkpoint: My results MAT file, two PNG figures, and two plot-value CSVs are saved.

## Troubleshooting

A repository 404 calls for checking the signed-in GitHub account and collaborator access. Small text pointer files call for git lfs pull. Missing functions call for checking the repository working directory and MATLAB path. A matrix assertion calls for checking the exact list, dimensions, finite entries and minimum trial length. An unexpected curve calls for checking response order and boundaries. A square matrix alone cannot establish sequence provenance.