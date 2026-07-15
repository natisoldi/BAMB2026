# Module 3 mini-project

There is one project, and everyone does it.

> **Train a policy on our demonstration dataset, so that it can actually do the task.**
> **On the last day, the best few policies in the class get deployed on the real arm.**

In [`tutorial_3b.ipynb`](./part2_imitation_learning/tutorial_3b.ipynb) you fit a policy to
demonstrations by maximum likelihood, and then watched a properly trained one drive the arm. Your
job is to close the gap between those two things.

## The data

**Train on the dataset we recorded on our own arm.** The Hub id will be given to you in the
session.

This matters more than it sounds. Do **not** train your final policy on the public
[`lerobot/svla_so101_pickplace`](https://huggingface.co/datasets/lerobot/svla_so101_pickplace)
data used in the tutorial. It is fine for prototyping while you get set up, but it was recorded
on a different arm, on a different table, with the camera somewhere else. A policy fit to it will
reach beautifully for where *their* cube used to be, and miss ours entirely, no matter how good
its held-out score is. Demonstrations do not transfer across scenes.

Everything loads through `LeRobotDataset`, exactly as in the tutorial.

## Where to start: the two upgrades

The tutorial's clone is deliberately minimal, and Section 4.1 named the two things it is missing.
Fixing either one is a real result. Fixing both is roughly what a deployable policy is.

**1. Give it eyes.** The clone's only input is its own joint angles, so when the cube moves it
reaches for the *average* cube. This is not fixable by making the network bigger — the
information is not in the input. Feed the camera in: a small convolutional network on downsampled
frames, alongside the joint angles. Trains on a laptop in minutes.

**2. Let it plan ahead.** The clone re-decides from scratch at every frame, on top of its own
accumulating errors, so it drifts into states no demonstrator ever visited. Predict a *chunk* of
the next ~30 actions at once and execute them, instead of one action at a time.

Both upgrades together, done properly, are essentially [ACT](https://arxiv.org/abs/2304.13705) —
and if you would rather stand on the shoulders of giants than write your own, **LeRobot will
train ACT for you on a free Colab GPU** (`lerobot-train`, a few hours, unattended). Doing that
well, and understanding *why* it works, is a perfectly good project. So is writing your own
smaller version and comparing.

Beyond those two, anything is fair game: how much data you actually need, throwing away the worst
demonstrations, data augmentation, or a loss that can represent more than one way of doing the
task (the demonstrations are not all alike, and mean-squared error quietly averages them).

## One trap worth hunting on purpose

Try adding **the previous action** to your policy's input. Your held-out error will improve, and
your policy will get *worse* — because copying your own last action is an excellent way to
predict the next one and a terrible way to do a task.

That is [causal confusion](https://arxiv.org/abs/1905.11979), and it is exactly the
over-imitating child from Section 1 of the tutorial: faithfully copying the part of the
demonstration that had nothing to do with why it worked.

It is also a warning about your scoreboard. **Held-out prediction error is not competence.** It
grades your policy frame by frame while handing it the true state every time, so its mistakes
never come home to roost — the one thing that cannot happen on a real robot. Watch out for
changes that improve the number and hurt the robot. Action chunking can even go the other way:
slightly worse held-out error, considerably better behaviour.

## Scope and deliverable

Twelve hours, three of you. One clear figure answering one clear question, plus a short
presentation. A negative result you can explain beats a positive result you cannot.

## Deployment day

Submit a trained policy by the deadline announced in the session. We cannot run thirty policies
on one arm, so we will pick **the best few** and run those in front of everyone on the last day.

Two rules, and they exist because a policy that has drifted off the demonstrations will happily
command a pose that damages a real motor:

- your policy must run at **15 Hz or better on CPU** — the arm cannot wait for you;
- your actions will be **clamped** to the joint ranges seen in the training data, and
  rate-limited. Do not rely on the clamp: a policy that needs it is not a policy that works.

Come and find us early with what you are trying, and we will help you scope it.
