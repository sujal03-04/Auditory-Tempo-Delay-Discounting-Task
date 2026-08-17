# Auditory Tempo Delay Discounting Task

A browser-based **jsPsych delay discounting task** adapted from Kirby et al. (1999) for a study examining whether auditory tempo influences subjective time perception and intertemporal decision making. The task presents repeated choices between smaller-sooner (SS) monetary rewards and larger-later (LL) rewards while participants are exposed to controlled auditory stimuli.

The project contains five standalone HTML versions: fast metronome, slow metronome, fast music, slow music, and a no-audio control. The behavioural task is kept consistent across conditions; only the auditory manipulation changes.

---

## Conditions

| File | Condition | Stimulus |
|---|---|---|
| `index180bpm.html` | Fast metronome | 180 BPM metronome |
| `index60bpm.html` | Slow metronome | 60 BPM metronome |
| `index180bpmMUSICVERSION.html` | Fast music | ~189 BPM classical music |
| `index60bpmMUSICVERSION.html` | Slow music | ~60 BPM classical music |
| `index control.html` | Control | No audio |

The metronome conditions use 180 BPM and 60 BPM. The music conditions use a fast Vivaldi track at approximately 189 BPM and a slow Bach track at approximately 60 BPM. The music tracks are classical and non-lyrical. The study uses a between-participant design, with each participant completing one condition only.

---

## Experiment Flow

```text
Audio check
    ↓
Participant ID
    ↓
Information sheet
    ↓
Consent
    ↓
Demographics + music preference
    ↓
Task instructions
    ↓
3 practice trials
    ↓
30-second time-perception task
    ↓
28 randomized delay-discounting trials
    ↓
CSV export
```

The delay discounting task contains 28 experimental trials plus practice trials. On each experimental trial, participants choose between:

```text
₹SS today
        OR
₹LL in X days
```

The trial order is randomized independently within each session. Reaction time and choice are recorded automatically.

---

## Setup

Keep the HTML files together with the jsPsych libraries, plugins, CSS, and audio files they reference.

The HTML files require:

```text
jspsych.js
jspsych.css
plugin-html-button-response.js
plugin-instructions.js
plugin-survey-html-form.js
```

and the corresponding audio files for the four auditory conditions.

### Experimenter name — required before data collection

The consent form currently contains:

```html
<p><strong>Experimenter:</strong> Experimenter</p>
```

Replace `Experimenter` with the actual experimenter's name **in all five HTML files** before collecting participants.

For example:

```html
<p><strong>Experimenter:</strong> Dr. Sujal Jain</p>
```

Changing the name in one HTML file does not change the other four.

---

## Running the Task

For testing, run the folder through a local web server rather than relying on `file://`.

From the task folder:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000/
```

and launch the HTML file corresponding to the assigned condition.

For participant data collection, the folder can be hosted on a web server and the appropriate condition-specific HTML page can be provided to each participant.

Before collecting data, test **all five versions** and confirm that the correct stimulus plays, the consent path works, and the CSV is generated.

---

## What the Task Records

Participant information is added to the jsPsych dataset as global properties:

| Variable | Description |
|---|---|
| `participant_id` | Participant-entered study ID |
| `age` | Age |
| `gender` | Gender response |
| `music_pref` | Musical training / preference response |
| `consent_name` | Name entered on consent form |
| `consent_date` | Date of consent |
| `consented` | `yes` / `no` |

Each delay-discounting trial records:

| Variable | Description |
|---|---|
| `trial_number` | Experimental trial number |
| `ss_amount` | Smaller-sooner amount |
| `ll_amount` | Larger-later amount |
| `ll_delay_days` | Delay of the larger-later reward |
| `choice` | `SS` or `LL` |
| `rt` | Reaction time in milliseconds |
| `k_trial` | Trial-level discounting coefficient for SS choices |
| `task_stage` | `"choice"` for experimental trials |

The separate time-perception task records `temporal_distance_rt`, measured in seconds. Participants are instructed to press the button when they believe exactly 30 seconds have passed without using a clock or counting aloud.

---

## How `k` Is Calculated

For trials where the participant chooses the smaller-sooner reward, the task calculates:

```text
k_trial = (LL - SS) / (SS × delay)
```

LL choices do not receive a `k_trial` value.

At the end of the experiment, positive `k_trial` values are log-transformed and averaged before being exponentiated:

```text
k_discounting = exp(mean(log(k_trial)))
```

Thus, the current implementation produces the **geometric mean of positive trial-level k values**.

The resulting:

```text
k_discounting
log_k_discounting
```

are added to the jsPsych dataset.

Higher `k` values represent steeper delay discounting: a stronger tendency to devalue delayed rewards and prefer immediate outcomes. Lower values represent relatively greater preference for delayed rewards.

---

## Data Output

At the end of the experiment, the task exports the records labelled:

```text
task_stage = "choice"
```

as a CSV using jsPsych's `localSave()` function.

The current output filenames are:

```text
180bpmDDtest.csv
60bpmDDtest.csv
180bpmMUSICVERSIONDDtest.csv
60bpmMUSICVERSIONDDtest.csv
0bpmDDtest.csv
```

The CSV is downloaded **locally through the participant's browser**. The supplied implementation does not upload responses to a remote database or backend server.

For data collection, raw CSV files should therefore be retrieved and stored securely by the researcher. Keep the original raw files unchanged and perform cleaning/analysis on copies.

---

## Recommended Data Workflow

```text
Participant completes task
        ↓
Browser generates CSV
        ↓
Researcher retrieves raw CSV
        ↓
Rename/archive using participant ID + condition
        ↓
Combine raw files for analysis
        ↓
Calculate participant-level measures
        ↓
Statistical analysis
```

The five conditions should remain identifiable in the final dataset. Do not combine metronome and music conditions unless the planned analysis specifically treats them as a common factor.

---

## Research Design

The tool implements two related auditory manipulations using the same behavioural task:

- **Metronome experiment:** fast (180 BPM), slow (60 BPM), and no-audio control.
- **Music experiment:** fast (~189 BPM), slow (~60 BPM), and no-audio control.

Participants are assigned to one auditory condition. The primary dependent variable is the delay discounting coefficient `k`; temporal distance, measured through the 30-second time-perception task, provides a second behavioural measure.

The task is adapted from the Kirby delay discounting paradigm, in which participants repeatedly choose between smaller immediate and larger delayed rewards.

---

## Files

| File | Description |
|---|---|
| `index180bpm.html` | 180 BPM metronome condition |
| `index60bpm.html` | 60 BPM metronome condition |
| `index180bpmMUSICVERSION.html` | Fast music condition |
| `index60bpmMUSICVERSION.html` | Slow music condition |
| `index control.html` | No-audio control |
| `jspsych.js` | jsPsych library |
| `jspsych.css` | jsPsych styling |
| `plugin-html-button-response.js` | Button-response plugin |
| `plugin-instructions.js` | Instruction-screen plugin |
| `plugin-survey-html-form.js` | Survey/form plugin |
| `*.mp3` | Auditory stimuli |

---

## Important Before Data Collection

- Replace the **Experimenter** placeholder in all five HTML files.
- Confirm the correct audio file is referenced by each condition.
- Test the control condition to ensure no auditory stimulus is played.
- Run a complete test participant through every condition.
- Verify that participant ID and demographic information appear in the output.
- Verify that the 28 experimental trials are exported.
- Verify that reaction times and `k_trial` values are populated correctly.
- Preserve raw CSV files without manual editing.
- Follow the approved study protocol and data-management requirements when handling participant information.

---

## Citation

### Task / Study

Jain, S., Goyal, S. & Sahay, A. (2026). *The impact of auditory tempo on consumer impulsivity: An empirical investigation* [Undergraduate thesis, FLAME University].

### Delay Discounting Paradigm

Kirby, K. N., Petry, N. M., & Bickel, W. K. (1999). Heroin addicts have higher discount rates for delayed rewards than non-drug-using controls. *Journal of Experimental Psychology: General, 128*(1), 78–87. https://doi.org/10.1037/0096-3445.128.1.78
