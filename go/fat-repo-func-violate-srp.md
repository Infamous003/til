# Fat repo functions violate SRP

## Smell
A repo function that touches tables owned by other modules/repos —
not just its own table. E.g. StudentRepo.Update() running raw SQL
against user_profiles, users, guardians, class_students, AND
user_exam_mode_subscription in one function.

## Why it's bad
- Hidden coupling: if guardians' schema/logic changes, you now have
  to go edit an unrelated student repo file to keep up. The
  dependency isn't visible from the module boundary.
- Violates SRP: repo function is doing service-layer orchestration
  (business logic, cross-entity coordination) while pretending to
  be a data-access function.

## Fix
Repo layer stays thin — one repo touches one table/entity it owns.
Orchestration, transaction boundaries, and business logic move to
the service layer, using each domain's own repo via dependency
injection.

## Before
```go
// BEFORE (bad) — repo function orchestrating across 5 tables it doesn't own
func (r *StudentRepo) Update(studentID, instituteID uuid.UUID, req dto.StudentUpdateInput) error {
	return r.db.Transaction(func(tx *gorm.DB) error {
		// checking if the student belongs to the institute
		var count int64
		if err := tx.Table("institute_members").
			Where("user_id = ? AND institute_id = ? AND deleted_at IS NULL", studentID, instituteID).
			Count(&count).Error; err != nil {
			return err
		}
		if count == 0 {
			return student.ErrStudentNotFound
		}

		// touching user_profiles — not this repo's table
		if err := tx.Table("user_profiles").
			Where("user_id = ?", studentID).
			Updates(profileUpdates).Error; err != nil {
			return err
		}

		// touching users — not this repo's table
		if err := tx.Table("users").
			Where("id = ?", studentID).
			Updates(userUpdates).Error; err != nil {
			return err
		}

		// touching guardians — not this repo's table
		if err := tx.Table("guardians").
			Where("user_id = ?", studentID).
			Updates(guardianUpdates).Error; err != nil {
			return err
		}

		// ...and so on for enrollment + exam mode subscription
		return nil
	})
}
```

Example:
- Student Service:
```go
type Service struct {
	tx                       *database.TxManager
	repo                     Repository
	userRepo                 user.Repository
	guardianRepo             user.GuardianRepository
	instRepo                 institute.Repository
	classRepo                class.Repository
	examModeSubscriptionRepo ExamModeSubscriptionRepository
	examModeRepo             ExamModeRepository
	userService              *keycloak.UserService
	adminCredentialsMailer   AdminCredentialsMailer
}

func (s *Service) Update(...) error {
    err := s.tx.WithTx(ctx, func(ctx context.Context) error {
        if err := s.userRepo.Update(); err != nil {
            return err
        }
        if err := s.guardianRepo.Update(); err != nil {
            return err
        }
        if err := s.classRepo.Update(); err != nil {
            return err
        }
        //....so on
	})
    //handle the error
}
```

Here each injected repo implements its own CRUD fucntionality and the StudentService can simply use them in a transaction. Each repo owns its own functionality. Notice there was no need for StudentRepository.Update() at all