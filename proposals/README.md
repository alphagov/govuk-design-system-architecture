# Proposals

Most changes to GOV.UK Design System, GOV.UK Frontend or GOV.UK Prototype Kit
can be implemented and reviewed via the normal GitHub pull request workflow
without raising a proposal.

Where changes would represent a substantial change to the architecture of a
product or the way the products work together we would expect a proposal to be
raised to gather feedback from the community and to ensure that the consequences
of the change are fully thought-out.

## Process

1. Create a new branch on this repository and copy `proposals/000-template.md`
   to a new file with the next free number, such as `001-my-proposal.md`.
2. Modify the template to add detail about your proposal.
3. Include any images etc in a separate directory with the same name as your
   proposal (`000-my-proposal`) and link to them.
4. Make a Pull Request (PR) for your branch.
5. Post a link to your PR in #govuk-design-system channel on Slack.
6. The Design System team and the wider community are able to discuss your
   proposal using both inline comments against your document and the general PR
   comments section. You will need to create a free GitHub account in order to
   comment.
7. As changes are requested and agreed in comments, make the changes in your RFC
   and push them as new commits.
8. Stay active in the discussion and encourage and remind other relevant people
   to participate. If you start a proposal it’s up to you to push it through the
   process and engage people.
9. Once consensus is reached and approvals given using the Github review system,
   the PR can be merged.

## Rejecting proposals

A proposal can be rejected by the Design System team. This can happen for many
reasons, including if a consensus isn’t reached, or if people agree that
rejecting it is the right thing to do. It might be that there are already plans
to address a given problem in a different way, or that it's just not the right
time to make the change. In this case the PR should be closed with a suitable
comment.

## Recording changes

Proposals should be considered an immutable record of a point in time. If we
want to change a decision that was made in a previous proposal, a new proposal
should be raised. If that proposal is accepted, the old proposal may be updated
to reflect the fact that it has been superseded, but should otherwise be left
intact.
